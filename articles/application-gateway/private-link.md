---
title: Azure Application Gateway Private Link (preview)
description: This article is an overview of Application Gateway Private Link.
services: application-gateway
author: greg-lindsay
ms.service: application-gateway
ms.topic: conceptual
ms.date: 11/02/2022
ms.author: greglin

---

# Application Gateway Private Link (preview)

Today, you can deploy your critical workloads securely behind Application Gateway, gaining the flexibility of Layer 7 load balancing features. Access to the backend workloads is possible in two ways:

- Public IP address - your workloads are accessible over the Internet. 
- Private IP address- your workloads are accessible privately via your virtual network / connected networks

Private Link for Application Gateway allows you to connect workloads over a private connection spanning across VNets and subscriptions. When configured, a private endpoint will be placed into a defined virtual network's subnet, providing a private IP address for clients looking to communicate to the gateway. For a list of other PaaS services that support Private Link functionality, see [What is Azure Private Link?](../private-link/private-link-overview.md).

:::image type="content" source="media/private-link/private-link.png" alt-text="Diagram showing Application Gateway Private Link":::

> [!IMPORTANT]
> Azure Application Gateway Private Link is currently in [public preview](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## Features and capabilities

Private Link allows you to extend private connectivity to Application Gateway via a Private Endpoint in the following scenarios:
-	VNet in the same or different region from Application Gateway
-	VNet in the same or different subscription from Application Gateway
-	VNet in the same or different subscription and the same or different Azure AD tenant from Application Gateway

You may also choose to block inbound public (Internet) access to Application Gateway and allow access only via private endpoints. Inbound management traffic still needs to be allowed to application gateway. For more information, see [Application Gateway infrastructure configuration](configuration-infrastructure.md#network-security-groups)

All features supported by Application Gateway are supported when accessed through a private endpoint, including support for AGIC.

## Private Link components

Four components are required to implement Private Link with Application Gateway:

- Application Gateway Private Link Configuration

   A Private link configuration can be associated with an Application Gateway Frontend IP address, which can then be used to establish a connection using a Private Endpoint. If there's no association to an Application Gateway frontend IP address, then the Private Link feature won't be enabled.

- Application Gateway Frontend IP address

   The public or private IP address where the Application Gateway Private Link Configuration needs to be associated to enable the Private Link Capabilities.

- Private Endpoint

   An Azure network resource that allocates a private IP address in your VNet address space. It's used to connect to the Application Gateway via the private IP address similar to many other Azure Services like Storage, KeyVault, etc., that provide private link access.

- Private Endpoint Connection

   A connection on Application Gateway originated by Private Endpoints. You can auto-approve, manually approve, or reject connections to grant or deny access.

## Limitations
- API version 2020-03-01 or later should be used to configure Private Link configurations.
- Static IP allocation method in the Private Link Configuration object isn't supported.
- The subnet used for PrivateLinkConfiguration cannot be same as the Application Gateway subnet.
- Private link configuration for Application Gateway doesn't expose the "Alias" property and must be referenced via resource URI.
- Private Endpoint creation doesn't create a \*.privatelink DNS record/zone. All DNS records should be entered in existing zones used for your Application Gateway.
- Azure Front Door and Application Gateway do not support chaining via Private Link.
- Source IP address and x-forwarded-for headers will contain the Private link IP addresses

## Next steps

- [Configure Azure Application Gateway Private Link](private-link-configure.md)
- [What is Azure Private Link?](../private-link/private-link-overview.md)
