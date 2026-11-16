# Link the Xiaomi event API to the actual module

MiCode's public include/miev/mievent.h contained five function bodies that
returned NULL/0 or did nothing. The in-tree qcom-hv-haptics module includes
xm-haptic.h and used those no-ops for event reporting. The current older
MiCode display implementation does not reference this API; newer stock
display/GPU modules do, but they are not interchangeable with this source.

Replace only the no-op bodies with declarations of the existing MiCode
public API. Current haptics callers keep the event pointer opaque and use
the allocation/add/write/destroy functions; they do not access its fields.
The implementation now comes from the original MIXM miev module.

The device passes the original ELF's five exported CRCs to core module
modpost via KBUILD_EXTRA_SYMBOLS, before external modules are built.
The stock adapter validates its imports against the completed source kernel
and compares the saved export table byte-for-byte with the original ELF.
No export CRC, vermagic, module enforcement or implementation is invented.

The selected public prototypes come from MiCode's supplied API. They have
not been validated by runtime calls. The newest ROM's boot kernel is 6.1.157
while the stock miev module was built against 6.1.68; its split BTF cannot be
resolved with that newer boot kernel's base BTF. Thus BTF inspection has not
independently proven the full module ABI. Real device loading and event
transport validation remain required.
