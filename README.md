# PE Madness

Portable Executable Parsing Library (PE Parsing Library)

## Description

Why?...Why not? :P

I made this library because I wanted to learn about parsing portable executable (PE) files in Rust even though there is a much better crate [pelite](https://docs.rs/pelite/latest/pelite/).

This library intentionally uses `windows-sys` crate and `unsafe` code and it's coded the way it is because when making [Reflective DLL Injection / Shellcode Reflective DLL Injection](https://github.com/memN0ps/srdi-rs) it is not possible to use heap allocated memory and anything dynamically sized must be put on a stack and all external functions can't be used until you resolve imports.

When self-loading or emulating `LoadLibraryA`, your code needs to be entirely self-contained and not rely on `libc` or any other (implicit) external API otherwise it becomes dependent on the loader. Also, instead of using the [obfstr](https://crates.io/crates/obfstr) crate to obfuscate strings, hashes are used instead to make dynamic analysis more difficult.


## Example

```rust
use pemadness::{
    dbj2_hash, get_export_by_hash, get_loaded_module_by_hash, get_process_id_by_name, is_wow64,
};

mod lib;

const NTDLL_HASH: u32 = 0x1edab0ed;

fn main() {
    println!("PE Madness!");
    println!("Process ID {}", get_process_id_by_name("explorer.exe"));

    let module_base = unsafe {
        get_loaded_module_by_hash(NTDLL_HASH).expect("Failed to get loaded module by name")
    };

    println!("Module Address {:p}", module_base);

    let export_hash = dbj2_hash("NtOpenProcess".as_bytes());
    let export_address = unsafe {
        get_export_by_hash(module_base, export_hash).expect("Failed to get export by hash")
    };

    println!("Export Address: {:p}", export_address);

    println!("Architecture x86_64: {}", is_wow64());
}
```

## References and Credit

* https://discord.com/invite/rust-lang-community (Rust Community #windows-dev channel)
* https://github.com/janoglezcampos/rust_syscalls/
* https://github.com/not-matthias/mmap/
* https://github.com/Ben-Lichtman/reloader/
* https://github.com/2vg/blackcat-rs/
* https://github.com/Kudaes/DInvoke_rs/
* https://github.com/zorftw/kdmapper-rs/
* https://github.com/MrElectrify/mmap-loader-rs/
* https://github.com/kmanc/remote_code_oxidation
* https://github.com/trickster0/OffensiveRust
* https://docs.microsoft.com/en-us/windows/win32/debug/pe-format
* https://crates.io/crates/pelite