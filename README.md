# Lab 8: Memory Management Simulation

A simulator, written in C, for a memory management unit that hands out memory from one contiguous partition. It supports three placement policies:

| Flag | Policy | How the free list is kept |
|------|--------|---------------------------|
| `-F` | First Fit (FIFO) | in the order blocks were freed (new blocks go to the back) |
| `-B` | Best Fit | ascending by block size |
| `-W` | Worst Fit | descending by block size |

Because the free list stays in policy order, allocation simply takes the **first** free block that is large enough.

Two linked lists track memory:

- **FREE_LIST**: unallocated blocks
- **ALLOC_LIST**: allocated blocks, sorted by starting address

## What was implemented

**`list.c`**
- `list_add_ascending_by_address`: inserts a block in order of its start address
- `list_add_ascending_by_blocksize`: inserts a block before the first block that is the same size or larger (block size = `end - start + 1`)
- `list_coalese_nodes`: walks an address-sorted list and merges physically adjacent blocks (`prev.end + 1 == cur.start`), freeing the absorbed node
- `list_is_in_by_pid`: checks whether any block belongs to a given PID
- `list_add_descending_by_blocksize`: now counts block size inclusively (`end - start + 1`) while walking the list, the same way the reference program does

**`mmu.c`**
- `allocate_memory`: finds a block of at least the requested size, removes it from the free list, assigns it to the PID, trims it to the requested size, adds it to the allocated list by address, and returns the leftover fragment to the free list according to the policy
- `deallocate_memory`: removes the PID's block from the allocated list, marks it free, and returns it to the free list according to the policy
- `add_to_free_list`: helper that picks the right insert for the active policy

Error messages:

```
Error: Memory Allocation <blocksize> blocks
Error: Can't locate Memory Used by PID:<pid>
```

## Input format

```
<partition size>
<pid> <blocksize>     allocate blocksize units to pid
-<pid> 0              free the memory owned by pid
-99999 0              coalesce / compact the free list
```

## Build and run

```sh
cd MMU
make
./mmu input0.txt -F     # First Fit / FIFO
./mmu input0.txt -B     # Best Fit
./mmu input0.txt -W     # Worst Fit
```

## Testing

For `input0.txt`, the output matches the provided reference binary (`mmu_ref`) byte for byte under all three policies. On 200 randomly generated workloads per policy, First Fit and Worst Fit match the reference every time. `mmu_ref` itself crashes (segfaults) on most Best Fit workloads, so for those runs the outputs were compared up to the crash: 199 of 200 match. The remaining one differs only a few steps before the reference crashes.
