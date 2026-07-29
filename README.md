# LDAP Directory Administration

A hands-on systems administration project covering OpenLDAP installation, directory tree design, LDIF-based entry management, and fine-grained access control (ACL) configuration on Debian Linux.

# Overview

This lab explores OpenLDAP (`slapd`) as a centralized directory service — from initial installation through building a multi-level organizational directory tree and implementing a complex, role-based access control policy. The project is split into two parts: **LDAP2** (server setup, querying, entry modification) and **LDAP3** (custom directory tree design + ACL policy implementation).

# Part 1 — LDAP Server Setup & Querying 

- Installed and configured `slapd` (OpenLDAP server) on Debian, reconfigured the base domain (`dc=debian,dc=net` → `dc=arlinux,dc=com`)
- Explored LDAP connection methods: `ldap://` (unencrypted TCP), `ldaps://` (TLS), `ldapi:///` (local Unix socket)
- Used `ldapsearch`, `ldapwhoami`, and `slapcat` to query the directory anonymously and as root via **SASL/EXTERNAL** authentication
- Distinguished server-level configuration (`cn=config`) from data-level entries (`olcRootDN` per database)
- Performed full **CRUD operations** on directory entries using LDIF and `ldapadd` / `ldapmodify` / `ldapdelete`:
  - Adding users and organizational units
  - Modifying single-value and multi-value attributes (`add`, `delete`, `replace`)
  - Renaming entries (`modrdn`, with `deleteoldrdn` behavior)
  - Moving entries between subtrees via `newsuperior`
- Integrated LDAP with **NSS** (Name Service Switch) so LDAP-defined users are recognized as local Linux system users (`getent passwd`)

# Part 2 — Directory Design & Access Control

Designed and implemented a 4-level organizational directory tree (a themed multi-school structure: Hogwarts, Beauxbatons, Durmstrang) using `inetOrgPerson`, with role-differentiated attributes:
- All persons: full name, email, phone number
- Students (incl. visitors): room number
- Professors: employee ID number

**Access control policy** (`olcAccess` rules) implementing a realistic multi-tier permission model:
- Admin: full write access to everything, including passwords
- `userPassword`: write-only by self or admin; unreadable by anyone else (defense against credential exposure)
- Self-service: each user can read their own record but only modify their password
- Role-based delegation: a designated administrator (Dumbledore) can modify most attributes of students within their own institution
- Attribute-scoped delegation: a professor (Minerva) can modify a specific attribute (`mail`) for a specific student group (Gryffindor) only
- Cross-institution read rules for professors vs. institution-scoped read rules for students
- Rule ordering and evaluation semantics (`dn.exact` vs `dn.regex` vs `dn.subtree` vs `dn.children`; first-match-wins evaluation; `by * break` vs `by * none`)
- Added descriptive metadata (`description`, `physicalDeliveryOfficeName`) and a new staff branch (`ou=staff`) with least-privilege visibility rules
- Added a new access rule allowing non-teaching staff limited read access (phone numbers only, for emergency contact) without exposing other attributes
- Verified rule-ordering conflicts (e.g., a broad early rule can inadvertently override a later, more restrictive one — first-match-wins)
- Added two further access rules: public read access to `description` attributes for OU-browsing, and strict admin/self-only visibility for `employeeNumber`

# Key Concepts Demonstrated

- OpenLDAP installation & reconfiguration (`slapd`, `dpkg-reconfigure`)
- LDIF format: `add`, `modify`, `modrdn`, `delete` operations
- SASL/EXTERNAL authentication vs. simple bind (`-x -W`)
- Distinguished Names (DN), Relative DNs (RDN), and directory tree hierarchy
- Fine-grained ACL design with `olcAccess` (attribute-level, subtree-level, and role-based rules)
- Rule evaluation order and precedence in OpenLDAP ACLs
- LDAP–NSS integration for system-level authentication

# Contents

- [`LDAP23.pdf`](./LDAP23.pdf) — Full lab report with terminal output, LDIF files, and rule-by-rule explanations for both LDAP2 and LDAP3
- `ldif/` — LDIF files used to build the directory tree and access control policy
- `magia_arboles.ldif` — Initial directory tree structure
- `ruless.ldif` — Core access control rules 
- `arbol2.ldif` — Directory tree extensions
- `add.ldif` — Additional ACL rules

# Tools & Environment

- **OS:** Debian GNU/Linux (Trixie)
- **Software:** OpenLDAP (`slapd` 2.6), `ldap-utils`, `ldapscripts`, `ldapvi`
- **Format:** LDIF (LDAP Data Interchange Format)

# Author

Kawtharul Jannah Mohd Sukki
