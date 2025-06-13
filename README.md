# Diseño de Base de Datos NoSQL para el Parque Nicole

## 🎯 Introducción

Este proyecto tiene como objetivo diseñar una base de datos NoSQL utilizando MongoDB para el Parque de Diversiones **Parque Nicole**. Se aplican los principios de **incrustación (embedding)** y **referencia (referencing)** para modelar las relaciones entre entidades como atracciones, zonas, visitantes, empleados, eventos y mantenimientos.

La elección entre incrustar o referenciar se basa en criterios como: frecuencia de acceso, tamaño del documento, integridad de datos y relaciones entre entidades.


## Analisis de entidades y Propuesta de Modelo de Datos
Entidad: Atracción
Nombre de la Colección: atracciones

Atributos Propuestos:
_id: ObjectId
nombre: String
tipo: String
descripcion: String
altura_minima: Number
capacidad: Number
estado: String
tiempo_espera_promedio: Number
zona_id: ObjectId
mantenimientos: Array de subdocumentos

🔗 Relaciones:
Entidad Relacionada: Zonas
Tipo de Relación: Muchos a Uno
Estrategia de Modelado: Referencia de _id de Zona en el documento de Atracción
Justificación: Permite modificar o consultar zonas sin duplicar datos en múltiples atracciones.

Entidad Relacionada: Mantenimiento
Tipo de Relación: Uno a Muchos
Estrategia de Modelado: Incrustación de registros de mantenimiento en la atracción
Justificación: Los mantenimientos están íntimamente ligados a una sola atracción y son registros históricos; es más eficiente almacenarlos embebidos.

Representacion en JSON:
{
  "_id": ObjectId("..."),
  "nombre": "Montaña Rusa Infernal",
  "tipo": "Montaña Rusa",
  "descripcion": "Alta velocidad con giros extremos",
  "altura_minima": 1.4,
  "capacidad": 20,
  "estado": "Operativa",
  "tiempo_espera_promedio": 40,
  "zona_id": ObjectId("..."),
  "mantenimientos": [
    {
      "fecha": "2025-06-01",
      "descripcion": "Cambio de frenos",
      "empleados_ids": [ObjectId("..."), ObjectId("...")],
      "costo": 500.00
    }
  ]
}

/*------------------------------------------------------*/

Entidad: Zona del Parque
Nombre de la Colección: zonas

Atributos Propuestos:
_id: ObjectId
nombre: String
descripcion: String

🔗 Relaciones:
Entidad Relacionada: Atracciones
Tipo de Relación: Uno a Muchos
Estrategia de Modelado: Las atracciones referencian a la zona
Justificación: Evita almacenar listas largas y actualizarlas en cascada si cambian las atracciones.

Representacion en JSON:
{
  "_id": ObjectId("..."),
  "nombre": "Zona Aventura",
  "descripcion": "Atracciones extremas y adrenalina"
}

/*------------------------------------------------------*/

Entidad: Visitante
Nombre de la Colección: visitantes

Atributos Propuestos:
_id: ObjectId
nombre: String
apellido: String
fecha_nacimiento: Date
email: String
historial_visitas: Array of Date
tickets_ids: Array of ObjectId

🔗 Relaciones:
Entidad Relacionada: Tickets
Tipo de Relación: Uno a Muchos
Estrategia de Modelado: Referencia de tickets en el documento de visitante
Justificación: Permite acceder a los tickets en forma independiente, sin duplicar información.

Representacion en JSON:
{
  "_id": ObjectId("..."),
  "nombre": "Laura",
  "apellido": "Mendoza",
  "fecha_nacimiento": "1990-07-12",
  "email": "laura@example.com",
  "historial_visitas": ["2025-05-15", "2025-06-01"],
  "tickets_ids": [ObjectId("..."), ObjectId("...")]
}

/*-----------------------------------------------*/

Entidad: Ticket
Nombre de la Colección: tickets

Atributos Propuestos:
_id: ObjectId
tipo: String (diario, anual, VIP)
precio: Number
fecha_compra: Date
fecha_validez: Date
visitante_id: ObjectId

🔗 Relaciones:
Entidad Relacionada: Visitante
Tipo de Relación: Muchos a Uno
Estrategia de Modelado: Referencia al visitante
Justificación: Un ticket pertenece a un único visitante; se prefiere mantenerlo separado para eficiencia de consulta.

Representacion en JSON:
{
  "_id": ObjectId("..."),
  "tipo": "VIP",
  "precio": 120.00,
  "fecha_compra": "2025-06-01",
  "fecha_validez": "2025-06-01",
  "visitante_id": ObjectId("...")
}

/*-----------------------------------------------*/

Entidad: Empleado
Nombre de la Colección: empleados

Atributos Propuestos:
_id: ObjectId
nombre: String
apellido: String
cargo: String
horario_trabajo: String
atracciones_ids: Array of ObjectId

🔗 Relaciones:
Entidad Relacionada: Atracciones
Tipo de Relación: Muchos a Muchos
Estrategia de Modelado: Referencia de atracciones en el documento del empleado
Justificación: Escalable, ya que un empleado puede trabajar en varias atracciones y viceversa.

Representacion en JSON:
{
  "_id": ObjectId("..."),
  "nombre": "Diego",
  "apellido": "López",
  "cargo": "Operador",
  "horario_trabajo": "09:00 - 17:00",
  "atracciones_ids": [ObjectId("..."), ObjectId("...")]
}

/*-----------------------------------------------*/

Entidad: Evento 
Nombre de la Colección: eventos

Atributos Propuestos:
_id: ObjectId
nombre: String
descripcion: String
horario: String
ubicacion: Object { tipo: String, id: ObjectId }
empleados_ids: Array of ObjectId

🔗 Relaciones:
Entidad Relacionada: Empleados
Tipo de Relación: Muchos a Muchos
Estrategia de Modelado: Referencia de empleados en el evento
Justificación: Flexibilidad y evita duplicación de información de empleados.

Entidad Relacionada: Atracción o Zona
Tipo de Relación: Uno a Uno polimórfico
Estrategia de Modelado: Objeto con tipo + ID (referencia polimórfica)
Justificación: Algunos eventos ocurren en una atracción, otros en una zona general.

Representacion en JSON:
{
  "_id": ObjectId("..."),
  "nombre": "Show de Magia",
  "descripcion": "Espectáculo familiar con magos y efectos especiales",
  "horario": "18:00",
  "ubicacion": {
    "tipo": "zona",
    "id": ObjectId("...")
  },
  "empleados_ids": [ObjectId("..."), ObjectId("...")]
}

/*-----------------------------------------------*/

Entidad: Mantenimiento
Nombre de la Colección: mantenimientos

Atributos Propuestos:
_id: ObjectId
fecha: Date
descripcion: String
costo: Number
empleados_ids: Array of ObjectId
atraccion_id: ObjectId

🔗 Relaciones:
Entidad Relacionada: Atracción
Tipo de Relación: Muchos a Uno
Estrategia de Modelado: Referencia
Justificación: Alternativa útil si los registros son muy frecuentes o extensos.

Representacion en JSON:
{
  "_id": ObjectId("..."),
  "fecha": "2025-06-01",
  "descripcion": "Mantenimiento preventivo",
  "costo": 300.00,
  "empleados_ids": [ObjectId("...")],
  "atraccion_id": ObjectId("...")
}




## Conclusiones
- Usé embedding en los casos donde los datos siempre se consultan juntos y no son demasiado grandes para que se consulte esa información al tiempo, ya que consultar una sola no seria muy util
- Usé referencing cuando los datos se repiten o están relacionados con varias entidades. Esto me sirvió en situaciones donde pueda haber riesgo duplicar información.
- Busqué un diseño que fuera flexible y fácil de escalar. Una de las ventajas de trabajar con MongoDB es que permite modificar el esquema sin muchas complicaciones, pensando en la escalabilidad de mi proyecto
- Intenté mantener un equilibrio entre rendimiento y organización, para que sea legible facilmente.

## Desafíos
- Uno de los retos más importantes fue decidir cuándo incrustar y cuándo referenciar. A veces no sabia cual usar, y mas en relaciones de muchos a muchos, como la de empleados y atracciones.
- También tuve que tener cuidado con los documentos incrustados que pueden crecer demasiado para que fuera legible y facil de entender.
- Al usar referencias, me encontré con que algunas consultas se volvieron más complejas, ya que requieren varias operaciones para obtener toda la información deseada.












