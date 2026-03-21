---
icon: windows
---

# Tools

### Recursos y referencias

| Recurso                 | URL                                                                                                                          |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| HackTricks AD           | https://book.hacktricks.xyz/windows-hardening/active-directory-methodology                                                   |
| PayloadsAllTheThings AD | https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md |
| ired.team               | https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse                                         |
| BloodHound docs         | https://bloodhound.readthedocs.io                                                                                            |
| Impacket ejemplos       | https://github.com/fortra/impacket/tree/master/examples                                                                      |
| TarLogic Kerberos guide | https://www.tarlogic.com/blog/how-kerberos-authentication-works/                                                             |
| AD Security blog        | https://adsecurity.org                                                                                                       |
| Harmj0y blog            | https://blog.harmj0y.net                                                                                                     |

> El flujo más eficiente: BloodHound para mapear el dominio → NetExec para verificar accesos y ejecutar ataques en masa → Impacket para explotar vía Linux → Mimikatz/Rubeus para ataques Kerberos desde Windows.

> Mantener siempre actualizados Impacket y NetExec — añaden nuevas técnicas y módulos constantemente.

> En entornos con EDR/AV, preferir técnicas OPSEC: wmiexec sobre psexec, tickets AES256 sobre RC4, evitar escribir mimikatz en disco (usar `Invoke-Mimikatz` o alternativas como `lsassy` / `nanodump`).
