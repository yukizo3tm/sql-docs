---
title: "Securing Replication Over the Internet"
description: "Securing Replication Over the Internet"
author: "MashaMSFT"
ms.author: "mathoma"
ms.date: 09/25/2024
ms.service: sql
ms.subservice: replication
ms.topic: how-to
ms.custom:
  - updatefrequency5
helpviewer_keywords:
  - "security [SQL Server replication], Internet"
  - "Internet [SQL Server replication], security"
monikerRange: "=azuresqldb-mi-current||>=sql-server-2016"
---
# Securing Replication Over the Internet
[!INCLUDE[sql-asdbmi](../../../includes/applies-to-version/sql-asdbmi.md)]
  Replication over the Internet can provide flexibility, particularly for mobile Subscribers, but you must configure Internet replication appropriately to ensure adequate security. [!INCLUDE[msCoName](../../../includes/msconame-md.md)] recommends using one of two techniques for securely sharing information over the Internet:  
  
-   Virtual private network (VPN)  
  
-   The Web synchronization option for merge replication  
  
## Virtual Private Network  
 Virtual private networks provide a simple and secure layered approach to replicating [!INCLUDE[ssNoVersion](../../../includes/ssnoversion-md.md)] data over the Internet. The VPN connection over the Internet logically operates as a Wide Area Network (WAN) link between the sites.  
  
 This is achieved by allowing the user to tunnel through the Internet or another public network using a protocol such as [!INCLUDE[msCoName](../../../includes/msconame-md.md)] Point-to-Point Tunneling Protocol (PPTP) available with the [!INCLUDE[msCoName](../../../includes/msconame-md.md)] Windows NT version 4.0 or [!INCLUDE[msCoName](../../../includes/msconame-md.md)] Windows 2000 operating system, or Layer Two Tunneling Protocol (L2TP) available with the Windows 2000 operating system. This process provides security and features similar to those available in a private network.  
  
 For more information about setting up a VPN, see the [!INCLUDE[msCoName](../../../includes/msconame-md.md)] Windows documentation.  
  
## Web Synchronization Through IIS  
 The web synchronization option for merge replication provides the ability to replicate data using the HTTPS protocol, which can be a convenient approach to replicating data through a firewall. For more information, see [Configure Web Synchronization](../../../relational-databases/replication/configure-web-synchronization.md) and [Security Architecture for Web Synchronization](../../../relational-databases/replication/security/security-architecture-for-web-synchronization.md).  
  
## Related content

- [Replication Security Best Practices](../../../relational-databases/replication/security/replication-security-best-practices.md)
- [View and modify replication security settings](../../../relational-databases/replication/security/view-and-modify-replication-security-settings.md)
