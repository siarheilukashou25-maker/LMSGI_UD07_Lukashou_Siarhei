# LMSGI_UD07_Lukashou_Siarhei

`report_invoice_willmantech.xml` esta 100% generada por la IA(Composer 2.5), ya que no se escribir en QWeb. 

**QWeb** es el motor de plantillas de Odoo. En el XML se escribe HTML normal y atributos especiales (`t-foreach`, `t-if`, `t-field`, etc.) que insertan datos reales de la base de datos al generar el informe.

El  archivo `report_invoice_willmantech.xml`  tiene  dos  partes:
1.  Plantilla  QWeb  (`<template id="report_invoice_willmantech">`)  —  maquetación  y  lógica  de  presentación.
2.  Registro  de  informe  (`ir.actions.report`)  —  registro  en  el  sistema:  «esta  plantilla  imprime  documentos  del  modelo  `account.move`  (facturas)».


### Como funciona:
### 1. El usuario pulsa «Imprimir» / «Descargar PDF»

Odoo localiza la acción  `action_report_invoice_willmantech`, ve el modelo  `account.move`  y la plantilla  `report_invoice_willmantech`.

### 2. El servidor pasa los datos a la plantilla

En la plantilla llega la lista de documentos  `docs`  (normalmente una o varias facturas). El bucle exterior:

    <t  t-foreach="docs"  t-as="doc">

Cada elemento es una factura; dentro de la plantilla se llama  `doc`  (número, fecha, líneas, totales, etc.).

### 3. Envoltorio de la página

    <t  t-call="web.html_container">

`t-call`  incluye la envoltura estándar de Odoo (estilos, estructura de página para PDF). Mi contenido se inserta dentro.

### 4. Enlace de campos —  `t-field`

Por ejemplo:

    <span  t-field="doc.name"/>
    
    ...
    
    <span  t-field="doc.date"/>

`t-field`  no es solo «mostrar texto». Odoo conoce el tipo de campo en el modelo y:

-   formatea la fecha según la localización;
-   el dinero — con moneda y separadores;
-   las relaciones (cliente) — mediante widgets (como  `partner_id`  con  `t-options`).

Indico el  nombre del campo en la BD  y el sistema se encarga de la presentación.

### 5. Condiciones —  `t-if`

Mostrar vencimiento o referencia solo si están rellenados:

    <p  t-if="doc.invoice_date_due"  class="mb-0">
    
    ...
    
    <p  t-if="doc.ref"  class="mb-0">

El bloque con  `t-if`  se renderiza  solo si  la condición es verdadera.

### 6. Bucle por líneas de factura —  `t-foreach`

Las líneas en Odoo son el campo  `invoice_line_ids`. La tabla se construye así:

    <t  t-foreach="doc.invoice_line_ids"  t-as="line">
    
    <tr>
    
    ...
    
    </tr>
    
    </t>

Por cada línea (producto/servicio) se crea una  `<tr>`: descripción, cantidad, precio, importe.

### 7. Columna inteligente «Descuento»

Primero un  recorrido por todas las líneas  sin pintar la tabla — solo comprobación:

    <t  t-set="show_discount"  t-value="False"/>
    
    <t  t-foreach="doc.invoice_line_ids"  t-as="line_check">
    
    <t  t-if="line_check.discount"  t-set="show_discount"  t-value="True"/>
    
    </t>

-   `t-set`  — define una variable en la plantilla (`show_discount`).
-   Si al menos una línea tiene descuento →  `show_discount = True`.

La cabecera y las celdas de descuento llevan  `t-if="show_discount"`: la columna  no aparece en el PDF  si no hay descuentos. Es más limpio que una columna vacía con ceros.

### 8. Totales

Al final, de nuevo  `t-field`: base imponible, impuestos (si hay),  total  `doc.amount_total`.

### 9. Registro  `ir.actions.report`

    <record  id="action_report_invoice_willmantech"  model="ir.actions.report">
    
    ...
    
    <field  name="model">account.move</field>
    
    <field  name="report_type">qweb-pdf</field>

Vincula la plantilla con el modelo de facturas y el tipo de salida  qweb-pdf. Tras instalar el módulo, en el menú de la factura puede aparecer «Factura WillmanTech».

## Entregable 2: Interoperabilidad de Datos (Extracción JSON/XML)

## `invoice_export.json`  (manual en Odoo)

1.  Ir a  Facturación → Facturas  y abrir una factura  confirmada.
2.  Anotar número, fecha y totales desde el formulario.
3.  Abrir el  cliente  y copiar ID, nombre y correo electrónico.
4.  Revisar las  líneas de factura  (descripción, cantidad, precio, subtotal).
5.  Montar un  array JSON  `[{...}]`  con  `customer`  y  `lines`  en un editor de texto.

## `invoice_ubl.xml`  (manual en Odoo)

1.  Confirmar la misma factura en Odoo (estado  Publicado).
2.  Instalar el módulo  EDI / UBL  si está disponible, o usar los datos de la factura como referencia.
3.  Copiar datos de  WillmanTech  (proveedor) y del  cliente  desde Contactos.
4.  Volcar líneas e importes en un XML  UBL 2.1  con namespaces  cac  y  cbc.
5.  Añadir  `CustomizationID`  y  `ProfileID`  PEPPOL  y guardar el fichero  `.xml`.
