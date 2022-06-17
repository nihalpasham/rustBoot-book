Below is the code for rust boot flow chart diagram in mermaid tool version 9.1.1 .

```sh
graph TD
A((Power On / <br> Board Reset)) -->|rustBoot Starts running| B(RustBoot Start)
style A fill:#f9f,stroke:#333, stroke-width:4px
B --> |opennig partitions|1[boot partition <br> Update partition]
1 --> 2{Check if <br>Boot Partition <br> is in BootInTestingStage<br> 0x10 }
2 --> |True i.e. the Updated partition <br> has not set the <br>Image state to success| D[Trigger the update <br> State the image state to <br>updating state <br> ]
D --> X[[Rollback to BootFirmware]]
2 --> |Flase| E{Check if <br>Update partition <br> is in <br> UpdateInUpdatingStage<br> 0x70}
E --> |True| F[Call RustBoot Update <br> Open all 3 partition]
F --> P{Verify the Integrity & <br> Authenticity of <br> Update Firmware}
P --> |True| S{Prevent Rollback <br> Check for the<br> firmware version <br> If <br>Updt FW Version < Boot FW}
P --> |False| K1(Panic!)
S --> |True| K4[Panic!]
S --> |False| O[Swap the <br>Update partition <br>with Boot partition <br> Set the Update image flag<br>to Testing state]
O --> W[[Jump to Updated Firmware <br> Validate the firmware's functionality <br>Set the Image flag to success state 0x00]]
E --> |False| G[Match boot partion type <br> 1. BootInNewState FF <br> 2. BootInSuccessState 00]
G --> |1| H{Verify the Interity and <br> Authenticity of the <br> Boot Firmware}
G --> |2  i.e. Allready <br> Updated firmware <br> is Present| I{Verify the Interity and <br> Authenticity of the <br> Boot Firmware}
H --> |True| j[[Jump to the BootFirmware]]
H --> |False| K2(Panic!)
I --> |True| L[[Jump to the Updated <br>firmware in <br>Boot Partition]]
I --> |False| K3(Panic!)
j --> M[[Trigger The update <br> Set the image state to <br> Updating 70]]
M --> z((Board Reset))
style z fill:#f9f,stroke:#333, stroke-width:4px
style K1 fill:#f16,stroke:#ff9f,stroke-width:2px,color:#fff,stroke-dasharray: 5 5
style K2 fill:#f16,stroke:#ff9f,stroke-width:2px,color:#fff,stroke-dasharray: 5 5
style K3 fill:#f16,stroke:#ff9f,stroke-width:2px,color:#fff,stroke-dasharray: 5 5
style K4 fill:#f16,stroke:#ff9f,stroke-width:2px,color:#fff,stroke-dasharray: 5 5
```