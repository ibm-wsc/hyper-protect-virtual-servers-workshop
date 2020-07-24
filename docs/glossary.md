This glossary of terms consolidates the terms for the Hyper Protect Virtual Servers offering in alphabetical order to help both customers and internal team members to understand our offering better.

## Appliance
IBM Secure Service Container based appliance provided by an Appliance Vendor. From Hosting Appliance perspective, it is the combination of IBM Secure Service Container and the Hyper Protect Servers Hosting Appliance.

## Appliance Administrator
The person administrating an appliance which includes tasks, such as configuring storage, or memory to the appliance or performing other configuration tasks through the API provided by SSC/the Hosting Appliance.

## Appliance Operational Data
Metrics, logs, appliance dump data, error logs, stack traces, kernel dump, etc.

## Appliance Protected Data
Appliance secrets, workload data, configuration data, settings, and other internal information stored by an appliance.

## Appliance Vendor
An internal, or external exploiter of SSC, packaging SSC into an appliance.

## BYOK
The abbreviation of Bring Your Own Key, which allows you to import your existing keys to Hyper Protect Crypto Services service instances that protect your keys with advanced encryption.

## BYOI
The abbreviation of Bring Your Own Image, which is a part of the IBM Hyper Protect Virtual Servers solution to support the development and deployment of your own container images on top of the Secure Service Container framework.

## Container
A runtime instance of an Open Container Image (OCI) image.

## Crypto administrator
The person who signs commands that are issued to the crypto units. Multiple administrators can be added to one crypto unit to increase security.

## Crypto units
A single unit that represents a hardware security module and the corresponding software stack that is dedicated to the hardware security module for cryptography.

## Data encryption keys
A cryptographic key used to encrypt data that is stored in an application.

## Datapool
Synonyms for Storage Pool.

## Docker Content Trust
Docker Content Trust (DCT) is a feature in the Docker containerization platform that provides the ability to use digital signatures for data sent to and received from remote Docker registries. These signatures allow client-side or runtime verification of the integrity and publisher of specific image tags.

Through DCT, image publishers can sign their images and image consumers can ensure that the images they pull are signed. Publishers could be individuals or organizations manually signing their content or automated software supply chains signing content as part of their release process.

## Docker registry
A Docker registry is a storage and content delivery system, holding named Docker images, available in different tagged versions.

## EP11
The abbreviation of Enterprise PKCS #11, which is specifically designed for customers seeking support for open standards and enhanced security. The EP11 library provides an interface very similar to the industry-standard PKCS #11 API.

## Envelope encryption
The process of encrypting data with a data encryption key and then encrypting the key with a root key that can be fully managed.

## FIPS 140-2 compliance
An organization’s adherence to the Federal Information Processing Standards (FIPS) 140-2 standards. This standard specifies the security requirements that will be satisfied by a cryptographic module utilized within a security system protecting sensitive information.  FIPS 140-2 outlines four levels of security: 

Level 1 - the lowest level of security. It covers the basic security features in a cryptographic module.

Level 2 - improves the physical security aspects of cryptographic modules. Examples of required physical security             measures are tamper-evident coatings, seals, or pick-resistant locks. Role-based authentication is included in this security level and ensures the operator accessing the module is authorized and is limited to their assigned actions. 

Level 3 - requires enhanced physical security, potentially with products available from the private sector. A multi-chip embedded module has to be contained in a strong enclosure that zeroizes critical security parameters when it is removed. This security level also uses identity-based authentication. 

Level 4 - requires tamper detection circuits to be able to detect any device penetration. This level is best for when cryptographic modules are in a physically unprotected environment that intruders can access. Modules must be able to detect fluctuations and zeroize themselves. Level 4 also protects against security compromises due to environmental conditions.

## FIPS 197 compliance
An organization’s adherence to Federal Information Processing Standards (FIPS) 197 which is an algorithmic standard that addresses the Advanced Encryption Standard (AES). AES specifies a FIPS-approved cryptographic algorithm that can be used to protect electronic data. The AES algorithm is a symmetric block cipher that can encrypt and decrypt information.

## GPG
The abbreviation of Gnu Privacy Guard, which is an open standard used for signing, encrypting, and decrypting texts with public and private keys to increase the security of communications.

## GREP11
Represents Enterprise PKCS #11 (EP11) APIs over gRPC calls, which is designed to be a stateless interface for cryptographic operations on cloud.

## gRPC
A modern open source high performance remote procedure call (RPC) framework that can connect services in and across data centers for load balancing, tracing, health checking, and authentication.

## Hardware security module
A physical appliance that provides on-demand encryption, key management, and key storage as a managed service.

## Hosting Appliance
A technical component within IBM Secure Service Container based appliances, providing the enablement for running Docker-based workloads.

## Image
The basis of the containers. An image is an ordered collection of root file system changes and the corresponding execution parameters for use within a container runtime.

## ISV
The abbreviation of Independent Software Vendor, who provides software solutions by developing and deploying containerized applications to the Secure Service Container partitions.

## Master key
An encryption key that is used to protect a crypto unit. The master key provides full control of the hardware security module and ownership of the root of trust that encrypts the chain keys, including the root key and standard key.

## Management Utilities
A set of applications to provide an alternate way of configuring Hyper Protect Crypto Services service instances using signature keys and master key parts stored on smart cards. 

## Management server
An x86 or s390x Linux management server used to run the commands provided by the Hyper Protect Virtual Servers offering, and administer the offering.

## Manifest
A manifest is generated by the Secure Build for audit purpose, which contains a copy of the github project cloned by the Secure Build container, a copy of the build log, and a build.json with the build status.

## Manifest public key
A manifest public key is used to verify the manifest generated by the Secure Build.

## Manifest private key
A manifest private key is used to sign the manifest during the Secure Build.

## Mutual TLS
Mutual authentication or two-way authentication refers to two parties authenticating each other at the same time. Mutual TLS is doing this over the TLS protocol.

## Namespace
A namespace such as ibmzcontainers that contains a number of unique images. For examples, the images include hpvsop-base, hyperpcons-worker, hyperpcons-riaas, and so on.

## Partition
A partition is the logic partition (LPAR) on the mainframe, and can be created by using the logic partitioning tools such as Hardware Management Console (HMC) or other logical partitioning tools.

## PKCS #11
The abbreviation of Public-Key Cryptography Standards #11, which defines a platform-independent API to cryptographic tokens, such as HSM and smart cards.

## Protected memory enclave
Defined private regions of memory whose contents are protected and unable to be read or saved to any process outside the enclave itself.

## Quotagroup
The storage assigned to a workload running on an appliance. The appliance administrator assigns FCP, or ECKD based storage to an appliance. He then creates quotagroups, representing parts of the underlying storage. He finally assigns quotagroups to workloads through the appliance API.

## Root keys
A symmetric wrapping key that is used for encrypting and decrypting other keys that are stored in a data service.

## Registry
A Registry is a hosted service containing repositories of container images that responds to the Registry API. For example, Docker Hub.

## Repository
A repository is a set of containerized images. A repository can be shared by pushing it to a registry server. Different images in the repository can be labeled using tags. For example, hpvsop-base.

## Repository definition files
An encrypted registration file or a repository definition file is used to register the repository, for authentication or validation reasons, such that a Hosting Appliance will trust that the image, when pulled from the registry, is authentic.

## Repository registration files
A cleartext Python or JSON format file, which is generated by the Secure Build container when the container is created. The JSON format repository registration file can be used as the direct input to generate an encrypted repository definition file.

## runc
A CLI tool for spawning and running containers according to the Open Container Initiative (OCI) specification.

## runq
An open-sourced hypervisor-based Docker runtime environment, which is based on runc to run regular containerized images in a lightweight KVM or Qemu virtual machine.

## s390x
The underlying architecture of IBM Z or LinuxONE mainframe.

## Secondary node
A cluster member that replicates the contents of the primary database in Hyper Protect DBaaS.

## Smart cards
An HSM that looks like a credit card with an embedded chip. The chip is capable of performing a limited set of cryptographic operations and is loaded with custom software.

## Smart card readers
A device that attaches to a workstation and allows the workstation to communicate with a smart card.

## Smart Card Utility Program
The application installed as part of the Management Utilities. It sets up and manages the smart cards used by the Trusted Key Entry (TKE) application.

## Signature key
An encryption key that is used by the crypto unit administrator to sign commands that are issued to the crypto unit.

## Standard key
Another resource in Hyper Protect Crypto Services to directly encrypt and decrypt data.

## Secure Build / Secure Image Build
The process of building the application code from a Git-like source repository into a container image for s390x architecture, signing the image by using the authentication keys, and publishing the image to the remote repository for later integration.

## Secure Service Container
A container framework based on the runq technology, that is supported by the IBM Z or LinuxONE servers.

## Secure Service Container partition
A type of logic partitions (LPARs) on the mainframe that runs the Secure Service Container framework.

## SSH
The abbreviation of Secure Shell, which is a cryptographic network protocol for operating network services securely over an unsecured network by using public and private keys.

## Storage Pool
A storage pool is a uniquely named collection of storage disks on which the appliance file system is mounted.

## System Administrator
This role includes the system administrator of a machine, storage administrators, and network administrators.

## tag
A tag is used to version images in a repository. For example, latest, 1.2.3.4-develop-a0d3aea, or s390x-develop-54a9045.

## Trusted Key Entry application
The application installed as part of the Management Utilities. It uses smart cards to load master keys in service instances and to perform other configuration tasks for service instances.

## Workload
The application and data provided and generated by a (running) Workload Image.

## Workload Data
Workload user or workload client data, workload logs, workload secrets stored in the appliance.

## Workload Image
A container-based image, provided by the Workload Vendor. An appliance only runs workload images which have been registered with the appliance through a repository definition file.

## Workload User
The end-user of a workload.

## Workload Vendor
The creator of a Docker image running on top of Hosting Appliance.
