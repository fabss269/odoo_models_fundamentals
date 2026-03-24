# Fundamentos para Crear Modelos en Odoo

## Tabla de Contenidos

1. [¿Qué es un Modelo en Odoo?](#1-qué-es-un-modelo-en-odoo)
2. [Tipos de Modelos](#2-tipos-de-modelos)
3. [Estructura Básica de un Módulo](#3-estructura-básica-de-un-módulo)
4. [Creación de un Modelo](#4-creación-de-un-modelo)
5. [Tipos de Campos](#5-tipos-de-campos)
6. [Atributos Comunes de los Campos](#6-atributos-comunes-de-los-campos)
7. [Campos Relacionales](#7-campos-relacionales)
8. [Campos Computados](#8-campos-computados)
9. [Métodos del ORM](#9-métodos-del-orm)
10. [Decoradores de la API](#10-decoradores-de-la-api)
11. [Restricciones (Constraints)](#11-restricciones-constraints)
12. [Herencia de Modelos](#12-herencia-de-modelos)
13. [Seguridad y Permisos](#13-seguridad-y-permisos)
14. [Ejemplo Completo](#14-ejemplo-completo)

---

## 1. ¿Qué es un Modelo en Odoo?

Un **modelo** en Odoo es una clase Python que representa una tabla en la base de datos PostgreSQL. Cada modelo hereda de una clase base proporcionada por el ORM (Object-Relational Mapper) de Odoo y define:

- Los **campos** (columnas de la tabla).
- La **lógica de negocio** (métodos y restricciones).
- El **comportamiento** de los registros (creación, lectura, actualización, eliminación).

Odoo usa su propio ORM construido sobre SQLAlchemy para abstraer las operaciones con la base de datos.

---

## 2. Tipos de Modelos

Odoo ofrece tres tipos principales de modelos:

| Clase Base           | Descripción                                                                 |
|----------------------|-----------------------------------------------------------------------------|
| `models.Model`       | Modelo estándar persistente. Los datos se guardan en la base de datos.      |
| `models.TransientModel` | Modelo temporal (wizards). Los registros se eliminan periódicamente.     |
| `models.AbstractModel` | Modelo abstracto. No crea tabla en BD; sirve como base para otros modelos.|

```python
from odoo import models

class MiModelo(models.Model):
    _name = 'mi.modulo.modelo'

class MiWizard(models.TransientModel):
    _name = 'mi.modulo.wizard'

class MiMixin(models.AbstractModel):
    _name = 'mi.modulo.mixin'
```

---

## 3. Estructura Básica de un Módulo

Un módulo de Odoo que define modelos sigue esta estructura de directorios:

```
mi_modulo/
├── __init__.py
├── __manifest__.py
├── models/
│   ├── __init__.py
│   └── mi_modelo.py
├── views/
│   └── mi_modelo_views.xml
├── security/
│   ├── ir.model.access.csv
│   └── security.xml
└── data/
    └── datos_demo.xml
```

**`__manifest__.py`** — Descriptor del módulo:

```python
{
    'name': 'Mi Módulo',
    'version': '16.0.1.0.0',
    'summary': 'Descripción breve del módulo',
    'author': 'Mi Empresa',
    'depends': ['base'],
    'data': [
        'security/ir.model.access.csv',
        'views/mi_modelo_views.xml',
    ],
    'installable': True,
    'application': True,
}
```

---

## 4. Creación de un Modelo

El bloque mínimo para definir un modelo en Odoo es:

```python
from odoo import models, fields

class Libro(models.Model):
    _name = 'biblioteca.libro'          # Nombre técnico del modelo (obligatorio)
    _description = 'Libro de Biblioteca' # Descripción legible (recomendado)

    name = fields.Char(string='Título', required=True)
    autor = fields.Char(string='Autor')
    fecha_publicacion = fields.Date(string='Fecha de Publicación')
    paginas = fields.Integer(string='Número de Páginas')
    activo = fields.Boolean(string='Activo', default=True)
```

### Atributos del modelo

| Atributo          | Descripción                                                          |
|-------------------|----------------------------------------------------------------------|
| `_name`           | Nombre técnico único del modelo (obligatorio para modelos nuevos).  |
| `_description`    | Descripción legible del modelo (recomendado).                        |
| `_inherit`        | Modelo(s) del que hereda.                                            |
| `_inherits`       | Herencia delegada (composición).                                     |
| `_order`          | Orden por defecto de los registros (ej. `'name asc'`).              |
| `_rec_name`       | Campo usado como representación textual del registro.                |
| `_table`          | Nombre personalizado de la tabla en la BD.                           |
| `_sql_constraints` | Restricciones a nivel de base de datos.                             |

---

## 5. Tipos de Campos

### Campos Básicos

```python
from odoo import models, fields

class EjemploCampos(models.Model):
    _name = 'ejemplo.campos'
    _description = 'Ejemplo de tipos de campos'

    # Texto
    nombre        = fields.Char(string='Nombre', size=100)
    descripcion   = fields.Text(string='Descripción')
    html_content  = fields.Html(string='Contenido HTML')

    # Numéricos
    cantidad      = fields.Integer(string='Cantidad')
    precio        = fields.Float(string='Precio', digits=(10, 2))
    total         = fields.Monetary(string='Total', currency_field='currency_id')

    # Fecha y Hora
    fecha         = fields.Date(string='Fecha')
    fecha_hora    = fields.Datetime(string='Fecha y Hora')

    # Booleano
    activo        = fields.Boolean(string='Activo', default=True)

    # Selección
    estado = fields.Selection(
        selection=[
            ('borrador', 'Borrador'),
            ('confirmado', 'Confirmado'),
            ('cancelado', 'Cancelado'),
        ],
        string='Estado',
        default='borrador',
    )

    # Binario (archivos/imágenes)
    imagen        = fields.Image(string='Imagen')
    documento     = fields.Binary(string='Documento', attachment=True)
    nombre_doc    = fields.Char(string='Nombre del Documento')
```

---

## 6. Atributos Comunes de los Campos

| Atributo      | Tipo     | Descripción                                                         |
|---------------|----------|---------------------------------------------------------------------|
| `string`      | `str`    | Etiqueta visible en la interfaz de usuario.                         |
| `required`    | `bool`   | Si es `True`, el campo es obligatorio.                              |
| `readonly`    | `bool`   | Si es `True`, el campo no es editable desde la UI.                  |
| `index`       | `bool`   | Si es `True`, se crea un índice en la columna de la BD.             |
| `default`     | valor / `lambda` | Valor por defecto al crear un registro.                    |
| `help`        | `str`    | Texto de ayuda (tooltip) en la UI.                                  |
| `copy`        | `bool`   | Si se copia el valor cuando se duplica el registro (default `True`).|
| `store`       | `bool`   | Si el campo se almacena en la BD (relevante en campos computados).  |
| `compute`     | `str`    | Nombre del método que calcula el valor del campo.                   |
| `depends`     | `list`   | Lista de campos de los que depende el campo computado.              |
| `tracking`    | `bool`   | Registra los cambios en el chatter del registro.                    |
| `groups`      | `str`    | Restringe el acceso al campo a determinados grupos.                 |

---

## 7. Campos Relacionales

### Many2one — Muchos a Uno

Un registro pertenece a otro (ej. un pedido pertenece a un cliente):

```python
cliente_id = fields.Many2one(
    comodel_name='res.partner',
    string='Cliente',
    required=True,
    ondelete='restrict',  # 'restrict', 'cascade', 'set null'
)
```

### One2many — Uno a Muchos

Un registro tiene varios registros relacionados (ej. un pedido tiene varias líneas):

```python
linea_ids = fields.One2many(
    comodel_name='pedido.linea',
    inverse_name='pedido_id',
    string='Líneas del Pedido',
)
```

### Many2many — Muchos a Muchos

Múltiples registros de un modelo se relacionan con múltiples registros de otro (ej. productos con etiquetas):

```python
etiqueta_ids = fields.Many2many(
    comodel_name='producto.etiqueta',
    relation='producto_etiqueta_rel',  # Nombre de la tabla intermedia
    column1='producto_id',
    column2='etiqueta_id',
    string='Etiquetas',
)
```

---

## 8. Campos Computados

Los campos computados calculan su valor a partir de otros campos en lugar de almacenarlo directamente:

```python
from odoo import models, fields, api

class PedidoLinea(models.Model):
    _name = 'pedido.linea'
    _description = 'Línea de Pedido'

    producto_id  = fields.Many2one('product.product', string='Producto')
    cantidad     = fields.Float(string='Cantidad', default=1.0)
    precio_unit  = fields.Float(string='Precio Unitario')
    subtotal     = fields.Float(
        string='Subtotal',
        compute='_compute_subtotal',
        store=True,      # Almacena el valor en BD (permite búsquedas y ordenado)
    )

    @api.depends('cantidad', 'precio_unit')
    def _compute_subtotal(self):
        for linea in self:
            linea.subtotal = linea.cantidad * linea.precio_unit
```

### Campo Computado con Inverso (lectura/escritura)

```python
    precio_con_iva = fields.Float(
        string='Precio con IVA',
        compute='_compute_precio_con_iva',
        inverse='_set_precio_con_iva',
    )

    @api.depends('precio_unit')
    def _compute_precio_con_iva(self):
        for rec in self:
            rec.precio_con_iva = rec.precio_unit * 1.21

    def _set_precio_con_iva(self):
        for rec in self:
            rec.precio_unit = rec.precio_con_iva / 1.21
```

---

## 9. Métodos del ORM

### Operaciones CRUD

```python
# CREAR
nuevo = self.env['biblioteca.libro'].create({
    'name': 'El Señor de los Anillos',
    'autor': 'J.R.R. Tolkien',
    'paginas': 1200,
})

# LEER / BUSCAR
libros = self.env['biblioteca.libro'].search([
    ('autor', '=', 'J.R.R. Tolkien'),
    ('paginas', '>', 500),
])

# Buscar y leer campos específicos (más eficiente)
datos = self.env['biblioteca.libro'].search_read(
    domain=[('activo', '=', True)],
    fields=['name', 'autor'],
    limit=10,
)

# ACTUALIZAR
libros.write({'activo': False})

# ELIMINAR
libros.unlink()
```

### Dominio de Búsqueda

Los dominios son listas de tuplas `(campo, operador, valor)`:

```python
domain = [
    ('estado', '=', 'confirmado'),
    ('fecha', '>=', '2024-01-01'),
    '|',                              # Operador OR (aplica a las dos siguientes condiciones)
        ('cliente_id.pais', '=', 'ES'),
        ('cliente_id.pais', '=', 'MX'),
]
```

| Operador  | Descripción                       |
|-----------|-----------------------------------|
| `=`       | Igual a                           |
| `!=`      | Distinto de                       |
| `>`       | Mayor que                         |
| `>=`      | Mayor o igual que                 |
| `<`       | Menor que                         |
| `<=`      | Menor o igual que                 |
| `in`      | Dentro de una lista               |
| `not in`  | Fuera de una lista                |
| `like`    | Contiene (sensible a mayúsculas)  |
| `ilike`   | Contiene (insensible a mayúsculas)|

### Métodos de Ciclo de Vida

```python
from odoo import models, fields, api

class MiModelo(models.Model):
    _name = 'mi.modelo'
    _description = 'Mi Modelo'

    name  = fields.Char(required=True)
    codigo = fields.Char(readonly=True)

    @api.model_create_multi
    def create(self, vals_list):
        """Se ejecuta al crear uno o varios registros."""
        for vals in vals_list:
            if not vals.get('codigo'):
                vals['codigo'] = self.env['ir.sequence'].next_by_code('mi.modelo')
        return super().create(vals_list)

    def write(self, vals):
        """Se ejecuta al actualizar un registro."""
        resultado = super().write(vals)
        # Lógica post-escritura
        return resultado

    def unlink(self):
        """Se ejecuta al eliminar un registro."""
        for rec in self:
            if rec.estado == 'confirmado':
                raise models.ValidationError(
                    'No se puede eliminar un registro confirmado.'
                )
        return super().unlink()
```

---

## 10. Decoradores de la API

| Decorador                    | Descripción                                                                |
|------------------------------|----------------------------------------------------------------------------|
| `@api.model`                 | El método no opera sobre un conjunto de registros (recordset vacío).       |
| `@api.model_create_multi`    | Reemplaza a `@api.model` en el método `create`; recibe una lista de vals.  |
| `@api.depends(*fields)`      | Marca un campo computado como dependiente de los campos indicados.         |
| `@api.onchange(*fields)`     | Ejecuta lógica en la UI cuando cambia alguno de los campos indicados.      |
| `@api.constrains(*fields)`   | Valida restricciones de negocio cuando cambian los campos indicados.       |

```python
from odoo import models, fields, api
from odoo.exceptions import ValidationError

class Reserva(models.Model):
    _name = 'hotel.reserva'
    _description = 'Reserva de Hotel'

    fecha_entrada  = fields.Date(string='Fecha de Entrada',  required=True)
    fecha_salida   = fields.Date(string='Fecha de Salida',   required=True)
    dias_estancia  = fields.Integer(string='Días de Estancia', compute='_compute_dias')
    habitacion_id  = fields.Many2one('hotel.habitacion', string='Habitación')
    precio_noche   = fields.Float(string='Precio por Noche')
    total          = fields.Float(string='Total', compute='_compute_total', store=True)

    @api.depends('fecha_entrada', 'fecha_salida')
    def _compute_dias(self):
        for rec in self:
            if rec.fecha_entrada and rec.fecha_salida:
                delta = rec.fecha_salida - rec.fecha_entrada
                rec.dias_estancia = delta.days
            else:
                rec.dias_estancia = 0

    @api.depends('dias_estancia', 'precio_noche')
    def _compute_total(self):
        for rec in self:
            rec.total = rec.dias_estancia * rec.precio_noche

    @api.onchange('habitacion_id')
    def _onchange_habitacion(self):
        if self.habitacion_id:
            self.precio_noche = self.habitacion_id.precio_base

    @api.constrains('fecha_entrada', 'fecha_salida')
    def _check_fechas(self):
        for rec in self:
            if rec.fecha_salida <= rec.fecha_entrada:
                raise ValidationError(
                    'La fecha de salida debe ser posterior a la fecha de entrada.'
                )
```

---

## 11. Restricciones (Constraints)

### Restricciones de Python (`@api.constrains`)

Permiten validaciones complejas con lógica Python (ver sección anterior).

### Restricciones SQL (`_sql_constraints`)

Se aplican directamente en la base de datos (más eficientes):

```python
class Producto(models.Model):
    _name = 'catalogo.producto'
    _description = 'Producto del Catálogo'

    name     = fields.Char(string='Nombre', required=True)
    referencia = fields.Char(string='Referencia', required=True)
    precio   = fields.Float(string='Precio')

    _sql_constraints = [
        # (nombre, definición_sql, mensaje_de_error)
        ('referencia_unica', 'UNIQUE(referencia)', 'La referencia del producto debe ser única.'),
        ('precio_positivo', 'CHECK(precio >= 0)', 'El precio no puede ser negativo.'),
    ]
```

---

## 12. Herencia de Modelos

### Herencia Clásica (`_inherit`)

Extiende un modelo existente añadiendo campos o métodos:

```python
from odoo import models, fields

class ResPartnerExtendido(models.Model):
    _inherit = 'res.partner'  # Extiende el modelo de contactos nativo de Odoo

    fecha_nacimiento = fields.Date(string='Fecha de Nacimiento')
    tipo_cliente = fields.Selection(
        selection=[('vip', 'VIP'), ('regular', 'Regular')],
        string='Tipo de Cliente',
        default='regular',
    )
```

### Herencia por Prototipo (`_inherit` + `_name` nuevo)

Crea un nuevo modelo copiando la definición de otro:

```python
class MiContacto(models.Model):
    _name = 'mi.contacto'
    _inherit = 'res.partner'
    _description = 'Mi Contacto Personalizado'
    # Hereda todos los campos de res.partner y crea una tabla nueva
```

### Herencia Delegada (`_inherits`)

Un modelo embebe otro usando una clave foránea (composición):

```python
class Empleado(models.Model):
    _name = 'mi.empleado'
    _inherits = {'res.partner': 'partner_id'}
    _description = 'Empleado'

    partner_id = fields.Many2one('res.partner', required=True, ondelete='cascade')
    cargo      = fields.Char(string='Cargo')
    salario    = fields.Float(string='Salario')
```

### Herencia mediante Mixin

Usa un `AbstractModel` para compartir campos y métodos entre varios modelos:

```python
class TimeStampMixin(models.AbstractModel):
    _name = 'timestamp.mixin'
    _description = 'Mixin de Auditoría'

    creado_por  = fields.Many2one('res.users', string='Creado por', readonly=True)
    modificado_en = fields.Datetime(string='Última Modificación', readonly=True)

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            vals['creado_por'] = self.env.user.id
        return super().create(vals_list)


class MiModeloAuditado(models.Model):
    _name = 'mi.modelo.auditado'
    _inherit = ['mi.modelo', 'timestamp.mixin']
    _description = 'Modelo con Auditoría'
```

---

## 13. Seguridad y Permisos

### Archivo `ir.model.access.csv`

Define los permisos CRUD por grupo para cada modelo:

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_biblioteca_libro_user,biblioteca.libro.user,model_biblioteca_libro,base.group_user,1,1,1,0
access_biblioteca_libro_manager,biblioteca.libro.manager,model_biblioteca_libro,base.group_system,1,1,1,1
```

| Columna         | Descripción                                                    |
|-----------------|----------------------------------------------------------------|
| `id`            | Identificador externo único del permiso.                       |
| `name`          | Nombre descriptivo.                                            |
| `model_id:id`   | Referencia al modelo (`model_` + nombre técnico con `.` → `_`).|
| `group_id:id`   | Grupo de seguridad al que aplica.                              |
| `perm_read`     | Permiso de lectura (1 = sí, 0 = no).                           |
| `perm_write`    | Permiso de escritura.                                          |
| `perm_create`   | Permiso de creación.                                           |
| `perm_unlink`   | Permiso de eliminación.                                        |

### Reglas de Registro (`ir.rule`)

Restringen el acceso a registros específicos mediante dominios:

```xml
<record id="rule_libro_propio" model="ir.rule">
    <field name="name">Sólo libros propios</field>
    <field name="model_id" ref="model_biblioteca_libro"/>
    <field name="domain_force">[('create_uid', '=', user.id)]</field>
    <field name="groups" eval="[(4, ref('base.group_user'))]"/>
</record>
```

---

## 14. Ejemplo Completo

A continuación se muestra un modelo completo que integra los conceptos anteriores:

```python
from odoo import models, fields, api
from odoo.exceptions import ValidationError


class BibliotecaLibro(models.Model):
    _name = 'biblioteca.libro'
    _description = 'Libro de Biblioteca'
    _order = 'name asc'
    _rec_name = 'name'

    # ── Campos básicos ───────────────────────────────────────────────────────
    name             = fields.Char(string='Título', required=True, tracking=True)
    isbn             = fields.Char(string='ISBN', size=13, index=True)
    sinopsis         = fields.Text(string='Sinopsis')
    paginas          = fields.Integer(string='Páginas')
    fecha_publicacion = fields.Date(string='Fecha de Publicación')
    portada          = fields.Image(string='Portada')
    activo           = fields.Boolean(string='Activo', default=True)

    estado = fields.Selection(
        selection=[
            ('disponible', 'Disponible'),
            ('prestado',   'Prestado'),
            ('baja',       'De Baja'),
        ],
        string='Estado',
        default='disponible',
        required=True,
        tracking=True,
    )

    # ── Campos relacionales ──────────────────────────────────────────────────
    autor_id         = fields.Many2one('res.partner', string='Autor',   required=True)
    editorial_id     = fields.Many2one('res.partner', string='Editorial')
    categoria_ids    = fields.Many2many(
        'biblioteca.categoria',
        string='Categorías',
    )
    prestamo_ids     = fields.One2many(
        'biblioteca.prestamo',
        'libro_id',
        string='Préstamos',
    )

    # ── Campos computados ────────────────────────────────────────────────────
    total_prestamos  = fields.Integer(
        string='Total de Préstamos',
        compute='_compute_total_prestamos',
        store=True,
    )

    @api.depends('prestamo_ids')
    def _compute_total_prestamos(self):
        for libro in self:
            libro.total_prestamos = len(libro.prestamo_ids)

    # ── Restricciones SQL ────────────────────────────────────────────────────
    _sql_constraints = [
        ('isbn_unico', 'UNIQUE(isbn)', 'Ya existe un libro con este ISBN.'),
    ]

    # ── Restricciones Python ─────────────────────────────────────────────────
    @api.constrains('paginas')
    def _check_paginas(self):
        for libro in self:
            if libro.paginas is not False and libro.paginas < 1:
                raise ValidationError('El número de páginas debe ser mayor a 0.')

    # ── Ciclo de vida ────────────────────────────────────────────────────────
    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            if not vals.get('isbn'):
                vals['isbn'] = self.env['ir.sequence'].next_by_code('biblioteca.libro')
        return super().create(vals_list)

    def action_prestar(self):
        """Cambia el estado del libro a 'Prestado'."""
        self.ensure_one()
        if self.estado != 'disponible':
            raise ValidationError('Solo se pueden prestar libros disponibles.')
        self.estado = 'prestado'

    def action_devolver(self):
        """Marca el libro como devuelto y disponible."""
        self.ensure_one()
        self.estado = 'disponible'
```

---

## Referencias

- [Documentación oficial de Odoo — ORM API](https://www.odoo.com/documentation/17.0/developer/reference/backend/orm.html)
- [Guía de desarrollo de módulos Odoo](https://www.odoo.com/documentation/17.0/developer/tutorials/server_framework_101.html)
- [Campos del ORM de Odoo](https://www.odoo.com/documentation/17.0/developer/reference/backend/orm.html#fields)
