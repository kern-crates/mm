# mm

Memory management implementation for a no_std environment

This module provides virtual memory management functionality, including:

+ Virtual memory areas (VMA) management
+ Page table operations
+ Memory mapping and unmapping
+ Process memory space management

Features

+ No standard library dependency
+ Support for file-backed mappings
+ Process memory isolation
+ Memory permission control

Core Components

+ MmStruct: Main memory management structure for a process
+ VmAreaStruct: Virtual memory area descriptor

## Examples

```rust
#![no_std]
#![no_main]

use core::panic::PanicInfo;
use mm::{MmStruct, VmAreaStruct};

#[macro_use]
extern crate axlog2;

#[no_mangle]
pub extern "Rust" fn runtime_main(cpu_id: usize, _dtb_pa: usize) {
    axlog2::init("debug");
    info!("[rt_mm]: ...");

    axhal::arch_init_early(cpu_id);
    axalloc::init();
    page_table::init();

    let mut mm = MmStruct::new();

    let va = 0;
    let vma = VmAreaStruct::new(va, 4096, 0, None, 0);
    mm.vmas.insert(va, vma);

    info!("[rt_mm]: ok!");
    axhal::misc::terminate();
}

#[panic_handler]
pub fn panic(info: &PanicInfo) -> ! {
    arch_boot::panic(info)
}

```

## Structs

### `MmStruct`

```rust
pub struct MmStruct {
    pub vmas: BTreeMap<usize, VmAreaStruct>,
    pub mapped: BTreeMap<usize, usize>,
    pub locked_vm: usize,
    /* private fields */
}
```

Represents the memory management structure for a process

#### Implementations

##### `impl MmStruct`

```rust
pub fn new() -> Self
```

Creates a new memory management structure with default values

```rust
pub fn deep_dup(&self) -> Self
```

Creates a deep copy of the current memory management structure including all virtual memory areas and page mappings

```rust
pub fn pgd(&self) -> Arc<SpinNoIrq<PageTable>>
```

Returns a reference to the page global directory

```rust
pub fn root_paddr(&self) -> usize
```

Returns the physical address of the root page table

```rust
pub fn id(&self) -> usize
```

Returns the unique identifier of this memory management structure

```rust
pub fn brk(&self) -> usize
```

Returns the current program break(heap end) location

```rust
pub fn set_brk(&mut self, brk: usize)
```

Sets a new program break(heap end) location

```rust
pub fn map_region(
    &self,
    va: usize,
    pa: usize,
    len: usize,
    _uflags: usize
) -> PagingResult
```

Maps a virtual address region to a physical address with specified flags

```rust
pub fn unmap_region(&self, va: usize, len: usize) -> PagingResult
```

Unmaps a region of virtual memory

### `VmAreaStruct`

```rust
pub struct VmAreaStruct {
    pub vm_start: usize,
    pub vm_end: usize,
    pub vm_pgoff: usize,
    pub vm_file: OnceCell<FileRef>,
    pub vm_flags: usize,
}
```

Represents a virtual memory area with its properties and permissions

#### Implementations

##### `impl VmAreaStruct`

```rust
pub fn new(
    vm_start: usize,
    vm_end: usize,
    vm_pgoff: usize,
    vm_file: Option<FileRef>,
    vm_flags: usize
) -> Self
```

Creates a new virtual memory area with specified parameters
