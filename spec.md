# reduct VM specification v0.1.0

This specification is versioned under [Semantic Versioning 2.0.0](http://semver.org/).

## Introduction

### Why?

Embedded scripting in documents is a very nice feature of many modern (and not-so-modern) platforms.
Unfortunately, these almost always have severe security holes, so they aren't universally implemented. (Think Javascript and NoScript.)

As well, some platforms (e.g. Firefox) don't handle infinite loops in Javascript particularly well.

What if a virtual machine could be defined that could be used in such a scenario without any security or denial-of-service issues?

### The goals

To define a bytecode and virtual machine that

* Are not turing complete.
* Are trivially proven to terminate. (i.e. perhaps implicit in the format)
* Are trivially proven to have a known bound on memory use. (i.e. perhaps specified in the format)
* Is simple enough that a bug-free implementation is possible and practical.
* Can be used for useful tasks.
* To be fully specified as to truly provide "write-once-run-anywhere" functionality.

### Not goals

These aren't goals of this specification

* To write provably correct programs.
* To run particularly fast.
* To define a programming language.
* To define a VM usable for entire programs.

These may be tackled at another time.

### Corollaries of the goals

* The bytecode format should be easily parsable.
* The bytecode format should be completely defined.

## Bytecode specification

### Endianness

In bytecode representation, all integers are big endian. Runtime functionality is defined to be independent of endianness.

### Fundamental format

A bytecode sequence is a sequence of octets with a known size, which must be expressible in 32 unsigned bits.

### Framing

When embedded within a larger format, a bytecode sequence may be stored in any necessary format.
Information, in some form, on the length of the bytecode sequence must be included.
Formats should include a checksum to guard against transmission errors or incorrect files.

When embedded within a format that does not provide sizing or checksums, the following format should be used:

    | SIZE    | ... BODY ... | CRC-32  |
    | 4 bytes | SIZE bytes   | 4 bytes |

The initial SIZE specifies the length of the BODY, meaning that the total size of this format is SIZE + 8.

The CRC is computed over the entire representation before it: this includes both the SIZE and the BODY.
The CRC algorithm used is be the same algorithm used by the PNG 1.2 specification.

**TODO: embed a description of that CRC**

When redact bytecode is stored directly in a file, it should have the following format:

    | MAGIC   | SIZE    | ... BODY ... | CRC-32  |
    | 4 bytes | 4 bytes | SIZE bytes   | 4 bytes |

This is the same as the above format for embedding, but with the MAGIC NUMBER at the start.
The MAGIC NUMBER is the byte sequence `82 65 67 0` (decimal), aka `RAC\0` or `52 41 43 00` in hex.

The MAGIC NUMBER is not included in the CRC.

### Well-formedness of sequences

A well-formed bytecode sequence follows all requirements set out in this document.

A valid implementation may abort a program at any point if any bytecode is not well-formed.
A valid implementation must abort a program if completing its execution would require executing code that is not well-formed.

### Validity of implementations

A valid implementation conforms to all rules specified within this document that are not marked as optional.

An implementation may be implemented in any way, so long as the behavior from the perspective of the program is precisely equivalent to that of this specification.
Precise timing details of execution are not considered specified by this specification.

### Within the sequence

A bytecode sequence is immutable. An implementation must not provide a method by which the program can modify the sequence.

A well-formed bytecode sequence must follow this format:

    | VERSION | REGISTERS | BUFFERSIZE | ... BYTECODE ... |
    | 2 bytes | 2 bytes   | 4 bytes    | varies           |

#### Versions

The VERSION specifies the version of this specification that it should be interpreted under.
A valid implementation may interpret any VERSION that it is familiar with, but must not interpret any VERSION which it cannot correctly interpret.
A valid implementation must interpret at least one VERSION correctly, and must document which VERSION(s) it can interpret correctly.

The VERSION field has the following bitstructure:

    | SCHEME | MAJOR  | MINOR  | PATCH  |
    | 1 bit  | 5 bits | 5 bits | 5 bits |

SCHEME must be set to zero. If this version scheme is replaced in a future specification, this first bit will be specified as one.

MAJOR, MINOR, and PATCH should be parsed as unsigned integers, and must be the Semantic Versioning version of this document to be valid under
this version of the specification. (See the top of this document for the current version.)

#### Registers

The REGISTERS field specifies the exact unsigned number of virtual registers that the implementation must make available to the program.

The reduct VM is a register-based virtual machine, not a stack-based virtual machine. Registers are numbered from zero. Each register stores one 64-bit number.

If the REGISTERS field contains the number 3, for example, then registers 0, 1, and 2 may be used by the program in appropriate locations. The program is not well-formed if its execution would require accessing any registers not included by the REGISTERS field.

#### Buffers

The BUFFERSIZE field specifies the size of the program's scratch buffer. This size is measured in bytes. The buffer may be used by certain operations that work with arrays of bytes.

#### Bytecode

The BYTECODE field contains a sequence of bytecode instructions and constants.

The program begins execution at the start of this field (known as index 0) and moves along the sequence, performing instructions.
The implicit register that stores the current index is known as the PC register.

Some instructions are encoded by a single byte. Other instructions are encoded by a sequence of bytes.
After executing either kind of instruction, the PC moves to the next instruction directly past the executed instruction.
(Unless, of course, the instruction modifies the current PC value.)

### Bytecode instructions

SPECIFICATION IS IN PROGRESS.

