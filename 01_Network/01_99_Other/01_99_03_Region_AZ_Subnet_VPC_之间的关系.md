
- AZ 是 Region 的子集，具有高可用性和容灾能力。
- **一个 VPC 属于一个 Region，但可以跨多个 AZ 部署资源（比如子网）**。
- Subnet : VPC 中的 IP 子集，是你在某个 AZ 中部署资源的基础网络单位。 **每个子网只能关联一个 AZ**，但一个 AZ 可以有多个子网。

Region（区域）
├── AZ a（可用区）
│   ├── Subnet 1（子网）→ 属于 VPC
│   └── Subnet 2（子网）→ 属于 VPC
├── AZ b（可用区）
│   ├── Subnet 3（子网）→ 属于 VPC
│   └── Subnet 4（子网）→ 属于 VPC
└── ...
     └── VPC（虚拟私有云）
