# Rpc client

```bash
# Users' RIDs-forced
for i in $(seq 500 1100); do
    rpcclient -N -U "" [IP_ADDRESS] -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo "";
done

# samrdump.py can also serve this purpose
```

### Enumeración de grupos

* Agrupaciones por: `enumdomgroups`
* Detalles de un grupo con: `querygroup <0xrid>`
* Los miembros de un grupo a través de: `querygroupmem <0xrid>`

### Enumeración de grupos de alias

* Alias agrupa por: `enumalsgroups <builtin|domain>`
* Miembros de un grupo de alias con: `queryaliasmem builtin|domain <0xrid>`

### Enumeración de dominios

* Dominios que usan: `enumdomains`
* El SID de un dominio se recupera a través de: `lsaquery`
* La información del dominio se obtiene mediante: `querydominfo`

### Enumeración de acciones

* Todas las acciones disponibles por: `netshareenumall`
* La información sobre un recurso compartido específico se obtiene con: `netsharegetinfo <share>`

### Operaciones adicionales con SID

* SID por nombre mediante: `lookupnames <username>`
* Más SID a través de: `lsaenumsid`
* El ciclo de RID para comprobar más SID se realiza mediante: `lookupsids <sid>`
