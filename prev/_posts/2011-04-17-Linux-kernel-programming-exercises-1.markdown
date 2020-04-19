---
layout: post
title: Linux kernel programming exercises 1
---
This is the first post of a series (i hope) about linux kernel development.
Despite there are [excellent][1] [books][2] that describe throughly both kernel and modules, there aren't many sources that provide programming exercises in the form __task-hint-solution__.

Purpose of this series of posts is to compensate for that lack. Source of exercises will be available on [github](https://github.com/rikiji) and commented here. Please note that modules/patches described here won't always be useful as they are just intended to increase linux kernel hacking experience.

Task
-------

The first proposed task is to develop a __kernel module__ aimed to check the __ARP table__ to detect when something fishy is going on. Requisites:

 * Solution has to be a __kernel module__, as it can be done without harming the kernel tree. ARP checking can be easily done in user space reading /proc/net/arp but that won't help in undestanding how the linux kernel works.
 * run checks at regular __intervals of time__ without any busy waiting.
 * Checks should detect __duplicated entries__ in the ARP table. That's an acceptable approssimation to detect arp spoofing.

Hints
-----

Some way to efficiently parse the kernel source tree is mandatory, oneliner below does the trick:

    find . -regex ".*\\.\([ch]\)" -exec grep -Hn $1 {} \;[/sh]
    Symbols accessible from modules are explicitly exported by following macros:
    
     * `EXPORT_SYMBOL(..)`
     * `EXPORT_SYMBOL_GPL(..)`
    
    so running the previous line of code within `linux-2.6.38.2/net/ipv4` results in a lot of matches, which grepped again for __arp__ lead to a reasonable amount of lines.
    [sh=plain]
    ./arp.c:119:EXPORT_SYMBOL(clip_tbl_hook);
    ./arp.c:202:EXPORT_SYMBOL(arp_tbl);
    ./arp.c:515:EXPORT_SYMBOL(arp_find);
    ./arp.c:718:EXPORT_SYMBOL(arp_create);
    ./arp.c:728:EXPORT_SYMBOL(arp_xmit);
    ./arp.c:754:EXPORT_SYMBOL(arp_send);
    ./arp.c:1160:EXPORT_SYMBOL(arp_invalidate);
    ./netfilter/arp_tables.c:61:EXPORT_SYMBOL_GPL(arpt_alloc_initial_table);
    ./netfilter/arp_tables.c:1900:EXPORT_SYMBOL(arpt_register_table);
    ./netfilter/arp_tables.c:1901:EXPORT_SYMBOL(arpt_unregister_table);
    ./netfilter/arp_tables.c:1902:EXPORT_SYMBOL(arpt_do_table);

`arp_tb` looks like a good place to start from.
    
Solution
------------ 
    
`arp_tbl` is an instance of a more general table `struct neigh_table`, which is used to keep track of associations between network and data link layers. Likely there's a set of functions ready to parse it laying somewhere in the kernel tree. `net/core/neighbour.c` can be found with the same combination of find and grep cited above. The function needed is `neigh_for_each`, which is also conveniently exported to modules. Function usage can be learnt by `./decnet/dn_neigh.c:535:`. Note that `nr_neigh_for_each` is not the same.
    
`neigh_for_each` requires a callback function that will be called once for every entry in the table. Optional argument is not required in this case.

    void neigh_handler(struct neighbour * n, void * null)
    {
      struct neigh_list_t *tmp;
      int found = 0;
      char hbuffer[HBUFFERLEN];
    
      /* search */
      list_for_each_entry(tmp, &neigh_list.list, list) {
        if(memcmp(n->ha,tmp->ha,n->dev->addr_len)==0) {
          format_hwaddr(n->ha, n->dev->addr_len, hbuffer);
          printk(KERN_ALERT "duplicated entry: %s\n", hbuffer);
          found = 1;
        }
      }
      
      /* add an entry */
      if(!found) {
        struct neigh_list_t * new_entry = (struct neigh_list_t *) kmalloc(sizeof(struct neigh_list_t), GFP_KERNEL);
        memcpy(new_entry->ha,n->ha,n->dev->addr_len);
        memcpy(new_entry->primary_key,n->primary_key,sizeof(u8 *));
        list_add(&(new_entry->list), &(neigh_list.list));
      }  
    }

Standard kernel data structures have been used to keep track of previous entries. A list is filled with IP and MAC addresses and then compared to each new neighbour entry. When a MAC address is not found in the current list it will be added at the end of the loop. When a MAC has already be seen instead, `"duplicated entry: XX:XX:XX:XX:XX:XX"` is printed and visible through __dmesg__.

To run this check every N seconds kernel __workqueues__ have been chosen. 

    static struct workqueue_struct * workq;
    static DECLARE_DELAYED_WORK(work, arp_tbl_check);
    ...
    workq = create_singlethread_workqueue("arp_tbl_check_wq");
    queue_delayed_work(workq, &work, HZ * 5);

For this case a simple single-thread single-work queue would suffice.`HZ * 5` sets the next run at approximately 5 seconds of jiffies away. Grab [full code](https://github.com/rikiji/arpcheck).

[1]: http://www.amazon.com/Linux-Kernel-Development-Robert-Love/dp/0672329468 "Linux Kernel Development"
[2]: http://lwn.net/Kernel/LDD3/ "Linux Device Drivers"
