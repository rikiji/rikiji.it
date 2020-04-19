---
layout: post
title: On ruby's garbage collector
---
Ruby 2.0 features a new garbage collection algorithm, called Bitmap Marking. To understand how this new approach works, a brief look at the ruby design is needed, starting with __ruby.h__.

All ruby objects are referenced through variables of the type `VALUE` in the C code: a `VALUE` is an unsigned integer that can be an immediate value (integer, float, symbol, true, false , nil) or a pointer to a `RBasic` structure. A `RBasic` structure holds two fields, `flags` and `klass`. There are many macros that can be used to check the type of an object just by accessing those flags, `BUILTIN_TYPE` is one of those and it is defined as follows:

    #define BUILTIN_TYPE(x) (int)(((struct RBasic*)(x))->flags & T_MASK)

The type of an object can be one of those in the enum `ruby_value_type`, some of them are:

    enum ruby_value_type {
	RUBY_T_NONE   = 0x00,

	RUBY_T_OBJECT = 0x01,
	RUBY_T_CLASS  = 0x02,
	RUBY_T_MODULE = 0x03,
	RUBY_T_FLOAT  = 0x04,
	RUBY_T_STRING = 0x05,
	RUBY_T_REGEXP = 0x06,
	RUBY_T_ARRAY  = 0x07,
	RUBY_T_HASH   = 0x08,
	RUBY_T_STRUCT = 0x09,
	RUBY_T_BIGNUM = 0x0a,
	RUBY_T_FILE   = 0x0b,
	RUBY_T_DATA   = 0x0c,
	RUBY_T_MATCH  = 0x0d,
	RUBY_T_COMPLEX  = 0x0e,
	RUBY_T_RATIONAL = 0x0f,
   	...

So how does it work in practice? Let's have a look at `RString`: it is a struct that holds a `RBasic` struct inside. When you want to check if a `VALUE` is a string, the first thing that `rb_type(VALUE obj)` does is to check if it is an immediate value. If not, it will be cast to `RBasic` and the flags will be accessed by `BUILTIN_TYPE`. The result can be compared to `RUBY_T_STRING`: if it matches, it is safe to cast (again) the `VALUE` to `RString`. Please note that the `RBasic` structure is still accessible with the new pointer, as it is included into `RString`!

    struct RString {
	struct RBasic basic;
	union {
	    struct {
		long len;
		char *ptr;
		union {
		    long capa;
		    VALUE shared;
		} aux;
	    } heap;
	    char ary[RSTRING_EMBED_LEN_MAX + 1];
	} as;
    };

RString has actually more features than RBasic, it can contain an array of characters directly (on a 32 bits system the max size is 12 + 1 bytes) or it can contain a reference to a string allocated elsewhere. Casting `RBasic` structures to other types is valid for all ruby objects, and this feature is exploited by the GC to allocate space easily.

Now that the basic strategies of `VALUE` handling used by the ruby interpreter are known, we can move on to __gc.c__. `RVALUE` is a wrapper structure used to hold any of the previously discussed ones. This is the base "unit" that will be used to allocate and free memory.

     typedef struct RVALUE {
	union {
	    struct {
		VALUE flags;		/* always 0 for freed obj */
		struct RVALUE *next;
	    } free;
	    struct RBasic  basic;
	    struct RObject object;
	    struct RClass  klass;
	    struct RFloat  flonum;
	    struct RString string;
	    struct RArray  array;
	    struct RRegexp regexp;
	    struct RHash   hash;
	    struct RData   data;
	    struct RTypedData   typeddata;
	    struct RStruct rstruct;
	    struct RBignum bignum;
	    struct RFile   file;
	    struct RNode   node;
	    struct RMatch  match;
	    struct RRational rational;
	    struct RComplex complex;
	} as;
    } RVALUE;

Ruby tracks allocated memory in structures called `heaps_slot`. Each of them includes a pointer to a linked list of `RVALUE` structures. Let's follow the flow of the code when a new object is created, I cut out not relevant lines:

    static VALUE
    newobj(VALUE klass, VALUE flags)
    {
	rb_objspace_t *objspace = &rb_objspace;
	VALUE obj;

	...

	obj = (VALUE)objspace->heap.free_slots->freelist;
	objspace->heap.free_slots->freelist = RANY(obj)->as.free.next;
	if (objspace->heap.free_slots->freelist == NULL) {
	    unlink_free_heap_slot(objspace, objspace->heap.free_slots);
	}

	MEMZERO((void*)obj, RVALUE, 1);
	objspace->total_allocated_object_num++;

	return obj;
    }
 
A `RVALUE` object is taken from a global freelist and the head of the list is shifted forwards.
This `obj` is a pointer to a memory area which is big enough to store any ruby structure: `RString`, `RArray`... When this list becomes empty, the GC is called and it will try to free some `RVALUE`. If not possible, a new `heaps_slot` (or more) will be allocated.

Now that is known how objects are allocated, we can inspect the two phases of the garbage collector. The phases are named __mark__ and __sweep__. In the first phase all the objects still reachable are marked. In the sweep phase all the objects not marked are identified and added to the freelist we mentioned before. Additionally, while traversing the lists all marks are reset to prepare the objects for the next GC round.

Starting from the function `garbage_collect`, we can dig into the mark phase, handled by the function `gc_marks`.  The real job is done by the function `gc_mark_stacked_objects`, which consists in a giant switch-case statement that handles all the `RVALUE` subtypes differently. In the sweep phase, which starts in `gc_sweep`, all the `RVALUE` objects without a mark are added to the freelist that is used to allocate new objects.

In the next post I'll focus on the improvements introduced by the garbage collector of Ruby 2.0 and maybe provide some benchmark result.