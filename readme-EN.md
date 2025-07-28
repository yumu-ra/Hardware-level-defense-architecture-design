# Hardware-Level Defense Architecture Design Based on FPGA and NPU Collaboration (Unimplemented Version)

## Project Overview

This project proposes a hardware-level defense architecture concept that uses FPGA as an "interaction gateway" and NPU focusing on "model computing". It aims to address underlying vulnerabilities through "functional splitting + dynamic reconstruction" at the hardware level, filling the gap in defending against hardware-level unknown vulnerability exploitation. This design only targets hardware defense; antivirus software is still necessary for dealing with viruses, trojans, and other software threats. As the initiator of the project, I hope to rely on the community to jointly implement and improve this concept.

## Core Architecture and Execution Process (Design Concept)

### Architecture Components



*   **FPGA (Interaction Gateway)**: Planned to include three core modules: DMA controller, instruction/behavior filtering, and communication interface. It is responsible for non-trusted zone interactions, such as receiving basic OS instructions, forwarding NPU detection results, and processing DMA requests.

*   **NPU (Computing Core)**: As the security core, it is planned to run a dedicated large model, responsible for sensitive instruction detection, memory analysis, and defense against unknown vulnerabilities, illegal instructions, and illegal operations.

*   **Auxiliary Components**: Planned to include a custom Linux kernel module, an external power supply, and an independent FPGA update trigger button.

### Execution Process



| Step | Operating Entity | Planned Specific Behaviors                                                                                                                                                                                                                                                  |
| ---- | ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | FPGA             | Planned to read memory segments via PCIe, vectorize data for use by the large model, capture instruction sequence segments written by the CPU to memory, extract opcode sequences and operand features, filter normal instructions, and retain candidate abnormal patterns. |
| 2    | NPU              | Planned to have the large model running on an independent NPU conduct in-depth checks on vectorized memory data and instruction features to defend against unknown vulnerabilities, illegal instructions, or illegal operations.                                            |
| 3    | FPGA             | Planned to send an interrupt signal to the OS through a custom kernel module after receiving the NPU's abnormal judgment result, triggering a pause in instruction execution.                                                                                               |

## Core Advantages of FPGA (Conceptual Design): Locking Vulnerability Risks



1.  **Dynamic Reconstruction Against Hardware Vulnerabilities**: In the concept, FPGA logic can be reconfigured in real-time through firmware updates, eliminating the need for physical replacement like traditional ASICs when vulnerabilities are exposed.

2.  **Isolating "Computing Core" and "Interaction Surface**: In the design, the NPU serves as the "security core" for model operation, responsible for sensitive instruction detection and memory analysis; the FPGA only handles "non-trusted zone interactions" — such as receiving basic OS instructions, forwarding NPU detection results, and processing DMA requests. Even if the FPGA is compromised (with a much lower probability than direct attacks on the NPU), the NPU's model and core logic remain in a physically isolated area (e.g., isolated via an internal bus) and cannot be directly tampered with. This "layered defense" is more risk-resistant than a single hardware component.

3.  **Adapting to "Hardware-Triggered Update" Mechanism**: The FPGA board is designed to have an independent "update trigger button". When pressed, the FPGA first enters "safe mode" (切断与 OS 的交互), completes reconstruction through an encrypted channel (such as reading new firmware from external storage via SPI), and the entire process does not rely on the OS, avoiding interference from malicious programs during updates. It also plans to implement "dual-zone verification" for firmware updates: the FPGA is internally divided into a "running zone" and a "backup zone". During updates, the new firmware is first written to the backup zone, and only after verifying the hash and signature are correct does it switch to the running zone, preventing the FPGA from becoming inoperable due to power failure or tampering during the update process. Additionally, it plans to minimize FPGA logic: the simpler the interaction logic of the FPGA, the smaller the vulnerability surface — for example, only retaining the three core modules of "address resolution, data forwarding, and verification".

## Defense Logic and Boundaries of the Large Model (Design Concept)

### Planned Defense Scope

In the design, the large model will identify the following abnormalities based on training data (including known illegal instruction samples, vulnerability exploitation features, and normal instruction distribution):



*   Instruction combinations that violate CPU architecture specifications (such as undefined opcodes);

*   Instruction sequences that deviate from normal program behavior patterns (such as frequent calls to privileged instructions in a short period);

*   Memory access instructions whose target addresses do not match the program segment permissions (such as user-mode instructions accessing kernel-mode memory);

*   Typical memory access patterns similar to "Meltdown" and "Spectre", and characteristic instruction sequences of shellcode (such as heap spraying, abnormal jumps triggered by stack overflows).

### Expected Limitations



*   Unable to parse the complete contextual semantics of instructions; for completely new instruction obfuscation techniques, the recognition rate depends on the generalization ability of the model;

*   For zero-day vulnerabilities, it can only improve the early warning probability by analyzing behavioral deviations outside the normal baseline, and cannot guarantee 100% recognition;

*   Judgments are based on probability, and attacks highly similar to normal operations may be missed, requiring combination with multi-layer log auditing;

*   Does not cover software-level vulnerabilities (such as application-layer buffer overflows) and needs to be used with system patches and antivirus software.

## Community Collaboration Guidelines (To Be Launched)

### Contribution Directions



*   Firmware Development: Participate in the development and optimization of FPGA firmware, improve dynamic reconstruction efficiency, and perfect the dual-zone verification mechanism.

*   Model Training: Participate in the training of the large model running on the NPU, expand the vulnerability sample library, and optimize instruction feature extraction rules.

*   Compatibility Adaptation: Participate in adapting to more hardware platforms and improving interaction logic with different OS.

*   Document Improvement: Participate in supplementing technical details and writing user tutorials and debugging guides.

### Participation Methods (To Be Clarified)



*   After the project is initialized, issues can be submitted to provide feedback, and code or documents can be contributed through Pull Requests.

*   After the project's Discussions section is opened, it can be used for technical exchanges.

## License Terms (Proposed)

This project plans to adopt the **GPLv3 + Non-Commercial Use Supplementary Terms** license:



*   Permission is granted to freely copy, modify, and distribute the design documents and code of this project.

*   Any modified versions must be released under the same license and retain the original author's information.

*   **Commercial use of this project and its derivative works is prohibited**, including but not limited to embedding in commercial hardware products, providing paid security services, etc.

*   Non-commercial use includes academic research, personal learning, open-source community collaboration, etc.

## Reference Projects

The "CPU Hardware Security Dynamic Monitoring and Control Technology" proposed by the Tsinghua University team is valuable for reference. This technology logically divides the CPU into an arithmetic engine and a monitoring control circuit, and real-time judges whether there is a hardware security threat by comparing the difference between the actual behavior of the hardware during CPU operation and the expected behavior given by the instruction set.

## Disclaimer

This design is currently only an architectural concept. Its expected value lies in filling the gap in the defense against hardware-level unknown vulnerability exploitation. Even if it is implemented in the future, it cannot replace existing security solutions. Users still need to combine multiple security measures to build a complete protection system.



***

This project is currently in the conceptual stage, and its implementation and development are highly dependent on community collaboration. We look forward to the participation of more developers and researchers to jointly turn the concept into reality, optimize defense logic, expand application scenarios, and make hardware-level defense technology more perfect.

> （注：文档部分内容可能由 AI 生成）