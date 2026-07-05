# Reflexión: Lo que aprendí haciendo el Agile Delivery Team

**Miguel Lara | 5 de julio de 2026**

---

## ¿Qué cambió en tu plan cuando separaste el trabajo en cuatro roles en vez de pensarlo "todo junto"?

Cuando empecé, veía freelancer-tools como "necesitamos time-tracking, facturación y pagos". Tres features cada uno con una lista de requisitos.

Esto cambió al separar el trabajo en 4 roles indpendientes

El Product Owner no preguntó "¿qué features necesitamos?" sino "¿qué problemas tienen los usuarios finales?" Eso llevó a épicas que hablan de outcomes: no es "implementar facturación", es "convertir horas en facturas sin pasar por Sheets → Siigo". Es el mismo trabajo, pero pensado desde el dolor del usuario, no desde un análisis técnico.

El Developer tenía que refinar una historia que tenía 13 puntos y criterios vagos. La presión de INVEST obligó a preguntas como: "¿qué exactamente es 'gestionar tareas'? Esas preguntas parecen rebuscadas, pero son la diferencia entre una historia que el developer no sabe como implenentar y una que el equipo puede construir en una semana.

El Architect no dibujó "la arquitectura perfecta". Justificó cada línea: ¿PostgreSQL? Porque dinero es exacto, ACID no es negociable. ¿OAuth2 + Google? Porque Daniela usa Google, y no quiero que pierda 10 minutos en signup. Cada decisión tiene una razón documentada.

Y el Scrum Master enfrentó realidad: "¿esto cabe en 20 puntos?" Descubrió que US-02 (precarga de factura) tenía una pregunta sin respuesta: "¿usamos tarifa actual o histórica?" Eso bloqueaba E-02 completa. No fue suerte; fue que el proceso expuso el problema antes de que el sprint empezara.

Sin separar roles, todo se habría realizado con el ojo del contratista (developer) y probablemente habríamos creado un sistema que no resolvía los problemas de los freelancers

---

## ¿Qué historia te costó más dejar lista según INVEST, y por qué? : US-02

US-02 fue difícil porque había una pregunta legítima por resolver. Un freelancer puede cambiar su tarifa entre enero y marzo. Si emite una factura en julio por horas de febrero, ¿usa la tarifa de febrero o la de hoy?

Es un problema real. No es un tecnicismo.

Se tenía que eligir entre ignorar la pregunta (viola Definition of Ready) o documentar un supuesto. Elegí lo segundo. Decidimos: "usamos tarifa actual, pero si cambió desde el time entry, mostramos un aviso."

Eso hizo la historia testeable. Pero se dejó claro: "Fase 2 refina esto con histórico de tarifas." No es una solución perfecta; es una decisión aceptada con sus trade-offs explícitos.

Un aprendizaje valioso que nos mostró que INVEST no es un checklist que rellenas. Es una presión que obliga a decidir. Sin esa presión, US-02 hubiera entrado al sprint con la pregunta sin responder, y al entegarla al desarrollador este no habría sabido como proceder.

De forma personal también me ayudó a aceptar que se pueden realizar sistemas en lso que no todo problema está perfectamente resuelto sino que se puede asumir soluciones para poder probar que estas soluciones son las correctas cuando se pruebe el sistema y luego pivotar de ser necesario

---

## ¿Para qué te serviría un gate de Definition of Ready en tu equipo real?

En el ejercicio, el gate rechazó la escritura de sprint-plan.md. Eso fue un "no" automático, que puede evitar que quien realiza el trabajo de por terminado su plan sin tener los requisitos que hacen que el desarrollo se ejecute sin peoblemas, lo que puede probocar retraso en la entrega del sistema

Además se da el caso en el que el developer (tech lead) puede dejar historias pendientes para completarlas luego de iniciar el sprint

Esto es muy imporante en un equipo real donde el delay en entrega puede causar perdidas de dinero

Sin gate:
- Lunes: "Sprint 1 tiene 20 puntos, US-02 está incluida."
- Miércoles: "Espera, US-02 tiene una open_question sin resolver."
- Viernes: Sprint fallido, y empieza el blame ("¿por qué no refinaste?").

Con gate:
- El Scrum Master intenta escribir el sprint plan.
- El gate: "No. US-02 no cumple DoR."
- Sprint realista desde el inicio.

Sin él gate el rigor depende del Scrum Master: si el tiempo de entrega se acerca puede dejar pasar cosas sin refinar. El gate aplica la regla igual a todas, siempre.

El gate detectó que E-02/E-03 no cabían en Sprint 1 porque US-02 tenía un bloqueante. Un equipo sin automatización pudo haber debatido eso hasta comprometerse con la  historia.
