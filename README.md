# Lab 03 Gestion de Turnos y Reservas en Clinica Municipal de Valencia

## Contexto
Una clinica municipal en Valencia ofrece citas medicas por especialidad. Cuando el agendamiento se maneja por llamadas o mensajes aparecen problemas comunes:
- doble cita del mismo medico en el mismo horario
- pacientes con datos incompletos o mal registrados
- cancelaciones de ultimo minuto sin trazabilidad
- necesidad de bloquear horarios por vacaciones, reuniones o emergencias
- alta demanda que requiere lista de espera

## Objetivo del proyecto
Implementar una API REST con Spring Boot y MySQL para gestionar citas medicas, incorporando:
- CRUD y relaciones
- prevencion de conflictos por rango horario
- autenticacion y roles
- bloqueos de agenda
- transiciones de estado con reglas
- lista de espera
- manejo de concurrencia y errores estandar JSON

## Alcance
Se trabajara con cinco entidades relacionadas:
- Doctor
- Paciente
- Cita
- BloqueoAgenda
- ListaEspera

## Dominio

## Entidad Doctor
### Campos
- id Long autoincrement
- nombre String
- especialidad String
- colegiado String
- activo boolean

## Entidad Paciente
### Campos
- id Long autoincrement
- docTipo String NIF NIE PASAPORTE
- docNumero String
- nombre String
- email String
- telefono String

## Entidad Cita
### Campos
- id Long autoincrement
- doctorId FK a Doctor
- pacienteId FK a Paciente
- fecha LocalDate yyyy-MM-dd
- horaInicio LocalTime HH:mm
- duracionMin int 15 30 45 60
- estado String SOLICITADA CONFIRMADA EN_ATENCION FINALIZADA CANCELADA NO_ASISTIO
- motivo String
- nota String opcional
- createdAt Instant
- updatedAt Instant
- version Long para optimistic locking

## Entidad BloqueoAgenda
### Campos
- id Long autoincrement
- doctorId FK a Doctor
- fecha LocalDate yyyy-MM-dd
- horaInicio LocalTime HH:mm
- horaFin LocalTime HH:mm
- motivo String

## Entidad ListaEspera
### Campos
- id Long autoincrement
- doctorId FK a Doctor
- fecha LocalDate yyyy-MM-dd
- pacienteId FK a Paciente
- prioridad int 1 a 5
- estado String ACTIVA NOTIFICADA ATENDIDA CANCELADA
- createdAt Instant

## Reglas de negocio

## Conflicto por rango horario
Una cita ocupa el rango:
- inicio = horaInicio
- fin = horaInicio + duracionMin

No se permite solapamiento con:
- otra cita del mismo doctor en estado SOLICITADA CONFIRMADA EN_ATENCION
- un bloqueo de agenda del mismo doctor

Regla de solapamiento
- hay conflicto si inicioNueva < finExistente y finNueva > inicioExistente

## Transiciones de estado
- SOLICITADA puede pasar a CONFIRMADA o CANCELADA
- CONFIRMADA puede pasar a EN_ATENCION o CANCELADA o NO_ASISTIO
- EN_ATENCION puede pasar a FINALIZADA
- FINALIZADA no puede cambiar
- CANCELADA no puede cambiar

## Politica de cancelacion
- si faltan menos de 2 horas para la cita, solo rol ADMIN puede cancelar
- si faltan 2 horas o mas, PACIENTE puede cancelar su cita

## Bloqueos de agenda
- un bloqueo no puede solaparse con otro bloqueo del mismo doctor
- no se permite crear bloqueo si ya hay citas en ese rango
- mejora avanzada para ADMIN
  - permitir crear bloqueo con un flag force que cancela citas afectadas y registra nota automaticamente

## Lista de espera
- si se intenta reservar y hay conflicto, se puede crear una entrada en lista de espera
- cuando una cita se cancela, se puede promover automaticamente al primer paciente en espera para ese doctor y fecha

## Seguridad

## Autenticacion
- Spring Security con JWT

## Roles
- PACIENTE puede crear y ver sus citas
- RECEPCION puede confirmar y gestionar agenda
- ADMIN puede gestionar todo incluyendo bloqueos forzados

## Reglas de acceso
- PACIENTE solo accede a recursos asociados a su pacienteId
- RECEPCION y ADMIN acceden a todos los recursos

## Endpoints sugeridos

## Doctores
- GET  /api/v1/doctores
- POST /api/v1/doctores  ADMIN

## Pacientes
- GET  /api/v1/pacientes
- POST /api/v1/pacientes

## Citas
- POST  /api/v1/citas  PACIENTE RECEPCION
- GET   /api/v1/citas/{id}
- GET   /api/v1/citas con filtros y paginacion
  - doctorId
  - pacienteId
  - fechaDesde
  - fechaHasta
  - estado
  - page
  - size
  - sort
- PUT   /api/v1/citas/{id}
- PATCH /api/v1/citas/{id}/confirmar
- PATCH /api/v1/citas/{id}/iniciar-atencion
- PATCH /api/v1/citas/{id}/finalizar
- PATCH /api/v1/citas/{id}/cancelar

## Bloqueos
- POST   /api/v1/bloqueos  RECEPCION ADMIN
- GET    /api/v1/bloqueos con filtros
  - doctorId
  - fechaDesde
  - fechaHasta
- DELETE /api/v1/bloqueos/{id}

## Lista de espera
- POST  /api/v1/espera
- GET   /api/v1/espera con filtros
  - doctorId
  - fecha
  - estado
- PATCH /api/v1/espera/{id}/cancelar
- POST  /api/v1/espera/promover  RECEPCION ADMIN

## Validaciones minimas

## Validaciones de paciente
- docTipo obligatorio
- docNumero obligatorio y consistente con docTipo
- nombre obligatorio minimo 3 caracteres
- email formato valido recomendado
- telefono formato valido recomendado

## Validaciones de cita
- doctorId debe existir si no 404 DOCTOR_NOT_FOUND
- pacienteId debe existir si no 404 PACIENTE_NOT_FOUND
- fecha obligatoria formato yyyy-MM-dd
- horaInicio obligatoria formato HH:mm
- duracionMin debe ser uno de 15 30 45 60
- motivo obligatorio minimo 5 caracteres
- estado se define por backend al crear como SOLICITADA
- al crear o actualizar validar conflicto por rango y bloqueos

## Validaciones de bloqueo
- doctorId debe existir
- fecha obligatoria
- horaInicio y horaFin obligatorias
- horaFin debe ser mayor a horaInicio
- no debe solaparse con otros bloqueos del mismo doctor

## Validaciones de lista de espera
- doctorId debe existir
- pacienteId debe existir
- fecha obligatoria
- prioridad entre 1 y 5
- estado se crea como ACTIVA

## Concurrencia y prevencion real de doble cita

## Estrategia recomendada con tabla de slots
Crear tabla CitaSlot:
- doctorId
- fecha
- horaSlot por ejemplo cada 15 minutos
- citaId
- indice unico doctorId fecha horaSlot

Al crear una cita dentro de transaccion:
- calcular slots que cubren la duracion
- insertar slots
- si falla la restriccion unica devolver 409 CITA_CONFLICT

## Alternativa con bloqueo pesimista
- consultar citas del doctor para la fecha con SELECT FOR UPDATE
- verificar solapamiento y luego insertar o actualizar
- requiere manejo cuidadoso de transacciones

## Errores estandar JSON
Responder siempre con:
```json
{
  "code": "SOME_CODE",
  "message": "Mensaje humano"
}
```

Codigos sugeridos:
- 400 VALIDATION_ERROR
- 401 UNAUTHORIZED
- 403 FORBIDDEN
- 404 DOCTOR_NOT_FOUND PACIENTE_NOT_FOUND CITA_NOT_FOUND
- 409 CITA_CONFLICT BLOQUEO_CONFLICT ESTADO_INVALIDO
- 422 BUSINESS_RULE_VIOLATION

## Checklist de aceptacion
- Proyecto levanta con ./mvnw spring-boot:run
- Swagger disponible en /swagger-ui/index.html
- MySQL conectado y migraciones aplicadas
- CRUD de doctores y pacientes operativo
- POST /api/v1/citas crea en estado SOLICITADA
- Conflicto por solapamiento devuelve 409 CITA_CONFLICT
- Bloqueos impiden crear citas en rango
- Transiciones de estado cumplen reglas y violaciones devuelven 409 ESTADO_INVALIDO
- Politica de cancelacion aplicada y violaciones devuelven 422 BUSINESS_RULE_VIOLATION
- PACIENTE no puede ver citas de otros pacientes
- Filtros con paginacion y orden funcionan
- Lista de espera se crea cuando hay conflicto
- Promocion desde lista de espera funciona tras cancelacion
