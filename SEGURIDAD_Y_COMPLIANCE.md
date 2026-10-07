# 🛡️ Marco de Ciberseguridad, Auditoría & Cumplimiento Normativo
### GestionA™ — Arquitectura de Protección Industrial
**Entidad Desarrolladora:** BetoGraf_Inc SpA  
**Nivel de Seguridad:** Grado Bancario / Misión Crítica Industrial  

---

## 1. 🔑 Autenticación FIDO2 / WebAuthn de Plataforma (Sin Contraseñas Débiles)

En entornos de terreno, las contraseñas compartidas escritas en papeles o anotadas en notas adhesivas representan una de las mayores vulnerabilidades operacionales. **GestionA™** elimina esta debilidad implementando estándares de criptografía asimétrica **FIDO2 / WebAuthn Level 3**:

* **Enlace con Hardware Seguro (Secure Enclave / TPM / Strongbox):**  
  La clave privada reside en el chip criptográfico dedicado del smartphone o computadora del técnico (`authenticatorAttachment: 'platform'`). Jamás viaja a través de la red ni puede ser extraída por malware.
* **Autenticación Biométrica 1-Touch:**  
  La validación se realiza mediante lectura biométrica local (huella dactilar física o reconocimiento facial 3D). Si la biometría presenta suciedad por grasa o polvo de planta, admite autenticación mediante el PIN local seguro del sistema operativo del dispositivo.
* **Desacoplamiento de Cuentas en la Nube:**  
  La llave biométrica se vincula exclusivamente a la sesión en GestionA, permitiendo que técnicos utilicen sus smartphones personales o corporativos sin exigir que la cuenta Google o Apple del teléfono coincida con su correo institucional.

---

## 2. 👥 Control de Acceso Basado en Roles (RBAC) y Mínimo Privilegio

La plataforma implementa una barrera estricta de 4 niveles jerárquicos:

```
[ SUPERADMIN ] ──► Control total de plataforma, auditoría global y parametrización
       │
[ SUPERVISOR ] ──► Control de cuadrillas, asignación de tickets, reportes y métricas
       │
[ COORDINADOR ] ──► Recepción de llamadas, despacho de órdenes y seguimiento básico
       │
  [ TÉCNICO ]   ──► Interfaz simplificada de terreno, órdenes propias y disponibles
```

* **Validación en Capa de Servidor:** Cada acción de actualización de datos, cierre de tickets o asignación valida criptográficamente la sesión activa antes de tocar la base de datos.
* **Aislamiento de Órdenes:** Un técnico únicamente puede visualizar detalles sensibles de las órdenes asignadas a su persona o aquellas explícitamente disponibles para ser tomadas en su planta asignada.

---

## 3. 📜 Cumplimiento de Estándares Internacionales

### A. Alineación con ISO 55000 (Gestión de Activos Físicos)
* **Hoja de Vida Unificada:** Cada intervención, mantención preventiva y falla queda ligada permanentemente al código único del activo (`EQ-[SIGLA]-[TIPO]-[CORRELATIVO]`).
* **Cálculo Auditado de Tiempos:** Tiempos de respuesta (MTTR) e intervalos medios entre fallas (MTBF) registrados cronológicamente sin posibilidad de manipulación manual.

### B. Estándar Criptográfico NIST SP 800-63B
* En accesos tradicionales por contraseña (fallback de contingencia), las credenciales se protegen con algoritmos de hashing adaptativos con sal única (`bcrypt` con factor de coste elevado), imposibilitando ataques de diccionario o tablas arcoíris.

### C. Cadena de Custodia Legal y Evidencia Digital
* **Metadatos Inmutables:** Cada fotografía adjuntada conserva su fecha, hora y folio asignado.
* **Firma Digitalizada:** La firma táctil del receptor se renderiza como imagen vectorial sellada en el PDF con hash único, generando un respaldo pericial idóneo para presentaciones ante compañías de seguros o tribunales laborales.

---

## 4. 🔒 Blindaje Client-Side contra Manipulación en Terreno

Para asegurar que los dispositivos utilizados en plantas industriales mantengan su enfoque exclusivo en la operación:
* **Ajuste de Pantalla Completa Ergonómico (100dvh):** Interfaz fluida sin barras de desplazamiento innecesarias ni desbordes táctiles.
* **Prevención de Zoom Accidental:** Desactivación de gestos de pellizco (*pinch-to-zoom*) y doble toque en campo para garantizar que los botones de acción crítica nunca queden fuera del alcance visual del técnico.
* **Bloqueo de Modos de Inspección:** Desactivación de menús contextuales y atajos de depuración para proteger la integridad visual y la sesión activa.

---

<div align="center">
  <sub>Documento confidencial para fines comerciales y de certificación técnica. © 2026 <b>BetoGraf_Inc SpA</b>.</sub>
</div>
