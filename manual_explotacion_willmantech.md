# Manual Técnico de Explotación — WillmanTech S.L.

  

_Basado en la Norma ISO/IEC/IEEE 26514:2022_

  
|Versión | 1.0 | 2026-05-21 |
|--|--| -- |
  

**Audiencias:** administradores de sistemas y usuarios finales (contabilidad, comercial).

  

---

  

## 1. Introducción y Arquitectura

  

Este manual describe la explotación del ERP de **WillmanTech S.L.** (Odoo 18).

  

-  **Módulos activados:** Facturación (`account`), Ventas, Contactos (`contacts`) y Web (`web`), para gestionar clientes, presupuestos y facturas.

-  **Artefactos del proyecto:** plantilla `report_invoice_willmantech.xml` (PDF), `invoice_export.json` (exportación), `invoice_ubl.xml` (factura electrónica UBL/PEPPOL).

-  **Topología lógica:** despliegue con **Docker Compose** — contenedor **odoo-web** (aplicación, puerto 8069) y contenedor **odoo-db** (PostgreSQL). El ERP se conecta al SGBD por el hostname de servicio `db`.

  

```text

[Navegador] → odoo-web:8069 → odoo-db:5432 (base odoo-db)

```

  

---

  

## 2. Guía de Instalación y Reinstalación

  

**Requisitos:** Docker, Docker Compose, 4 GB RAM.

  

1. Clonar el repositorio y situarse en la carpeta raíz.

2. Crear `.env` con variables del SGBD:

-  `POSTGRES_DB` (ej. `odoo-db`)

-  `POSTGRES_USER` (ej. `odoo`)

-  `POSTGRES_PASSWORD`

3. Arrancar servicios: `docker compose up -d`

4. Abrir `http://localhost:8069`, crear la base de datos e instalar **Facturación**.

5.  **Reinstalación limpia:**  `docker compose down -v` y repetir pasos 2–4 (pierde todos los datos; hacer backup antes).

  

---

  

## 3. Seguridad y Control de Acceso

  

Modelo **RBAC** (roles en **Ajustes → Usuarios**):

 
| Rol | Privilegios |
|--|--|
| **Administrador** | Configuración, módulos, usuarios y acceso total |
| **Contable** | Crear, confirmar y consultar facturas e informes financieros |
| **Comercial** | Clientes y presupuestos; solo lectura en facturas publicadas |


  

**Contraseñas:** mínimo 12 caracteres, con mayúsculas, números y símbolos. Activar 2FA para administrador y contable. No subir `.env` al repositorio.

  

---

  

## 4. Procedimiento de Backup y Restauración

  

**Backup de la base de datos:**

  

```bash

docker  exec  -t  odoo-db  pg_dump  -U  odoo  -d  odoo-db  -F  c  >  backup_willmantech_$(date  +%F).dump

```

  

**Backup del filestore** (adjuntos y ficheros Odoo):

  

```bash

docker  cp  odoo-web:/var/lib/odoo  ./backup_filestore_$(date  +%F)

```

  

**Restauración** (con Odoo detenido: `docker compose stop web`):

  

```bash

docker  exec  -i  odoo-db  pg_restore  -U  odoo  -d  odoo-db  --clean  <  backup_willmantech_YYYY-MM-DD.dump

docker  compose  start  web

```

  

---

  

## 5. Flujo Operativo de Facturación e Informes

  

1.  **Facturación → Facturas → Nuevo** — seleccionar cliente y líneas.

2.  **Confirmar** — la factura pasa a estado *Publicado*.

3.  **Imprimir** / **Factura WillmanTech** — genera el PDF.

  

**Pipeline del informe:**

  

```text

Datos (account.move) → QWeb (XML) → HTML → wkhtmltopdf → PDF

```

  

-  **QWeb** evalúa `report_invoice_willmantech.xml` (`t-field`, `t-foreach`, `t-if`).

-  **wkhtmltopdf** convierte el HTML con estilos al PDF final entregado al navegador.

---