# GestionA™ — Modelos de Arquitectura para Escalabilidad Multi-Empresa
**Documento Técnico & Estratégico de Evolución de Producto**  
**Autor:** BetoGraf_Inc SpA  
**Fecha:** Octubre 2026  
**Estado:** Propuesta de Arquitectura & Roadmap  

---

## 1. Resumen Ejecutivo y Visión

Actualmente, **GestionA™** opera de forma altamente especializada y exitosa como la plataforma de gestión operativa, mantenimiento y auditoría ejecutiva para **Anvic Security** (en conjunto con **Toledos SpA**), administrando infraestructura crítica en clientes como *Correos de Chile* (CEP Renca y CTP Quilicura), *Cial Alimentos*, *Estadio Monumental* y *Orsan Valle Grande*.

Dado el valor operativo, la trazabilidad criptográfica y la robustez técnica demostrada por el software, surge la oportunidad de comercializar esta solución a **otras empresas del rubro de seguridad privada, mantenimiento industrial, climatización, facilities o control de accesos**.

Este documento detalla los **dos modelos arquitectónicos viables** para habilitar que múltiples empresas utilicen GestionA™, evaluando ventajas, requerimientos técnicos, modelos de negocio y la estrategia de transición recomendada para **BetoGraf_Inc SpA**.

---

## 2. Los Dos Modelos de Arquitectura Posibles

```
                      ┌─────────────────────────────────────────┐
                      │      GESTIONA™ (BetoGraf_Inc SpA)       │
                      └────────────────────┬────────────────────┘
                                           │
                 ┌─────────────────────────┴─────────────────────────┐
                 ▼                                                   ▼
┌─────────────────────────────────┐                 ┌─────────────────────────────────┐
│           MODELO 1              │                 │           MODELO 2              │
│      Marca Blanca Dedicada      │                 │       SaaS Multi-Tenant         │
│     (Single-Tenant Aislado)     │                 │      Nativo Centralizado        │
├─────────────────────────────────┤                 ├─────────────────────────────────┤
│ • 1 Instancia x Cliente         │                 │ • 1 Plataforma Compartida       │
│ • BD 100% aislada (Supabase)    │                 │ • BD única particionada (RLS)   │
│ • Branding y dominio propio     │                 │ • Subdominios automáticos       │
│ • Máxima seguridad bancaria     │                 │ • Economía de escala máxima     │
│ • Setup + Fee Mensual de soporte│                 │ • Suscripción mensual / anual   │
└─────────────────────────────────┘                 └─────────────────────────────────┘
```

---

### MODELO 1: Marca Blanca Dedicada (Single-Tenant Aislado)
*Instancia de infraestructura y base de datos independiente por cada empresa cliente.*

#### 2.1.1. Cómo Funciona
En este modelo, cada nueva empresa que contrata el servicio recibe un despliegue autónomo:
- **Dominio propio:** `gestiona.anvic.cl`, `operaciones.empresa2.cl`, o subdominios de BetoGraf como `empresa2.gestiona.betograf.cl`.
- **Base de Datos Dedicada:** Cada cliente tiene su propio proyecto de Supabase (PostgreSQL) o base de datos dedicada. Ningún dato de la Empresa A comparte servidor con la Empresa B.
- **Parametrización por Entorno:** La aplicación se parametriza dinámicamente mediante variables de entorno (`.env`), eliminando cualquier texto o logo estático del código fuente.

#### 2.1.2. Requisitos Técnicos para Habilitarlo
1. **Motor de Branding Dinámico (`Config Provider`):**
   - Extraer logos, razones sociales, RUTs, correos de soporte y certificados a variables de configuración o tabla de `ConfiguracionGlobal`:
     * `NEXT_PUBLIC_EMPRESA_NOMBRE="Anvic Security"`
     * `NEXT_PUBLIC_EMPRESA_SLOGAN="Seguridad Integral"`
     * `NEXT_PUBLIC_EMPRESA_RUT="76.xxx.xxx-x"`
     * `NEXT_PUBLIC_EMPRESA_LOGO_URL="/logos/anvic.png"`
     * `NEXT_PUBLIC_PARTNER_NOMBRE="Toledos SpA"`
     * `SUPERADMIN_EMAIL="soporte@betograf.cl"`
2. **Plantillas PDF Dinámicas:**
   - Hacer que [PdfTemplate.tsx](file:///e:/pruebas/proyecto_atencion_ticket/ticket-system/src/components/PdfTemplate.tsx) y [ReportesEjecutivoPdfTemplate.tsx](file:///e:/pruebas/proyecto_atencion_ticket/ticket-system/src/components/ReportesEjecutivoPdfTemplate.tsx) lean el logo y los textos de cabecera desde la configuración del entorno en lugar de constantes estáticas.
3. **Pipeline de Despliegue Automatizado (CI/CD):**
   - Repositorio base con branches de cliente o despliegue multi-entorno en Vercel / Docker donde cada nuevo cliente se aprovisiona en minutos vinculando una nueva base de datos.

#### 2.1.3. Ventajas
- **Aislamiento Absoluto de Datos:** Cero riesgo de filtración o fuga de información entre competidores directos. Es el modelo preferido por clientes corporativos exigentes, mineras o entidades financieras.
- **Personalización sin Afectar a Otros:** Si un cliente requiere un campo específico en sus órdenes de trabajo o un flujo personalizado, se puede implementar sin alterar a los demás.
- **Despliegues Independientes:** Actualizaciones y mantenimientos pueden programarse en horarios diferenciados por cliente.

#### 2.1.4. Desafíos
- Mayor sobrecarga operativa al actualizar la versión base a través de múltiples proyectos.
- Costo de infraestructura directo por cliente (amortizado en la tarifa de cobro).

---

### MODELO 2: SaaS Multi-Tenant Nativo Centralizado
*Una sola plataforma compartida donde múltiples empresas coexisten de manera segura y transparente.*

#### 2.2.1. Cómo Funciona
En este modelo, existe una sola instancia del software y una única base de datos PostgreSQL. Toda la información está estrictamente etiquetada y filtrada por la empresa propietaria:
- Los usuarios inician sesión en un portal único (o vía subdominio tipo `anvic.gestiona.cl`, `protec.gestiona.cl`).
- Al autenticarse, el sistema reconoce a qué empresa (`Organizacion`) pertenece el usuario.
- Todos los paneles (Dashboard, Tickets, Instalaciones, Equipos, Técnicos, Reportes y Auditoría) muestran única y exclusivamente los datos de su propia organización.

#### 2.2.2. Modificaciones Requeridas en la Base de Datos (Prisma)
1. **Nueva Entidad `Organizacion` (Tenant):**
```prisma
model Organizacion {
  id              String         @id @default(uuid())
  nombre          String         // Ej: "Anvic Security", "Protec Alarmas SpA"
  rut             String         @unique
  slogan          String?        // Ej: "Seguridad Integral"
  logoUrl         String?
  subdominio      String         @unique // ej: "anvic", "protec"
  colorPrimario   String?        @default("#0ea5e9")
  plan            String         @default("ENTERPRISE") // BASIC, PRO, ENTERPRISE
  activa          Boolean        @default(true)
  createdAt       DateTime       @default(now())
  updatedAt       DateTime       @updatedAt

  usuarios        Usuario[]
  instalaciones   Instalacion[]
  tickets         Ticket[]
}
```

2. **Campo `organizacionId` Transversal:**
   - Agregar `organizacionId String` en los modelos clave:
     * `Usuario`
     * `Instalacion`
     * `Equipo`
     * `Ticket`
     * `AuditoriaLog`
3. **Seguridad a Nivel de Filas (Row-Level Security - RLS):**
   - Implementar RLS en PostgreSQL o Prisma Extensions que inyecte automáticamente `{ where: { organizacionId: session.organizacionId } }` en cada consulta de lectura, creación, actualización o eliminación.
4. **Superadmin Global de BetoGraf:**
   - Rol `GLOBAL_SUPERADMIN` (perteneciente a `soporte@betograf.cl`) con privilegios para ver métricas consolidadas, crear nuevas empresas, suspender accesos por falta de pago y gestionar la salud global del ecosistema vía `/api/health`.

#### 2.2.3. Ventajas
- **Economía de Escala:** Un solo servidor, una sola base de datos, un solo despliegue para atender a cientos de empresas.
- **Actualizaciones Inmediatas:** Cada mejora, corrección de seguridad o nueva funcionalidad queda disponible instantáneamente para todos los clientes.
- **Onboarding Automatizado:** Posibilidad de registrar una nueva empresa cliente en segundos de manera automatizada.

#### 2.2.4. Desafíos
- Requiere una reingeniería profunda del esquema Prisma, queries y endpoints existentes para garantizar el aislamiento estricto de datos.
- Mayor responsabilidad en auditoría de seguridad para evitar que un fallo lógico exponga datos cruzados entre empresas.

---

## 3. Matriz Comparativa de Modelos

| Criterio | Modelo 1: Marca Blanca Dedicada | Modelo 2: SaaS Multi-Tenant Nativo |
| :--- | :--- | :--- |
| **Tiempo de Salida al Mercado** | **Inmediato (1 a 2 semanas)** | Mediano Plazo (6 a 8 semanas) |
| **Riesgo de Fuga de Datos** | **0% (Aislamiento físico/lógico completo)** | Bajo (Dependiente de RLS y middleware) |
| **Complejidad de Código** | **Baja** (Parametrización vía `.env`) | Alta (Refactor de todo el modelo relacional) |
| **Percepción del Cliente** | **Exclusividad / Sistema a medida** | Software empaquetado / Compartido |
| **Modelo de Facturación** | Setup de Implementación + Mantención Fija | Suscripción recurrente mensual/anual |
| **Flexibilidad de Personalización** | **Total por cliente** | Estandarizada para todos |
| **Escalabilidad de Mantenimiento** | Media (N despliegues) | **Alta (1 solo despliegue)** |

---

## 4. Recomendación Estratégica para BetoGraf_Inc SpA

### Estrategia Progresiva en Dos Fases:

```
┌─────────────────────────────────────────────────────────────┐
│ FASE 1 (Corto Plazo): MARCA BLANCA PARAMETRIZADA            │
│ • Parametrizar variables de entorno de branding.            │
│ • Vender como solución dedicada para 2-3 nuevos clientes.   │
│ • Cobro de setup inicial + mantenimiento mensual.           │
│ • Financia el desarrollo de la siguiente fase sin riesgo.   │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ FASE 2 (Mediano Plazo): EVOLUCIÓN A SAAS MULTI-TENANT       │
│ • Crear tabla Organizacion e integrar RLS.                  │
│ • Migrar clientes a subdominios centralizados.              │
│ • Habilitar panel de control SaaS para BetoGraf_Inc SpA.    │
└─────────────────────────────────────────────────────────────┘
```

1. **Fase 1 (Inmediata / Marca Blanca Dedicada):**
   - Parametrizar el código actual de modo que ningún nombre de empresa ni logo esté cableado a fuego (*hardcoded*).
   - Utilizar el endpoint `/api/health` implementado como telemetría centralizada para monitorear todas las instancias desplegadas desde el panel de control de BetoGraf.
   - Ofrecer la solución a nuevas empresas con un modelo de cobro de **Setup Inicial de Instalación + Fee Mensual de Servidores y Soporte**.

2. **Fase 2 (Escalabilidad Masiva / Multi-Tenant):**
   - Una vez validados múltiples clientes y consolidada la tracción comercial, ejecutar la migración a arquitectura Multi-Tenant con particionamiento de base de datos y onboarding automatizado.

---

## 5. Checklist Técnico Inmediato para Habilitar Nuevas Empresas

- [x] **Separación de Instalaciones y Clientes:** Soporte nativo para agrupar instalaciones por empresa (`Correos de Chile`, `Cial Alimentos`, `Estadio Monumental`, `Orsan Valle Grande`).
- [x] **Reportes Oficiales Segregados:** Selector inteligente en `/admin/reportes` para emitir auditorías por cliente o instalación específica con folios corporativos dedicados.
- [x] **Monitoreo & Centro de Control:** Endpoint `/api/health` operativo con diagnóstico de base de datos, memoria, latencia y telemetría de fallos.
- [ ] **Variables de Entorno de Branding:** Desacoplar constantes corporativas en `PdfTemplate` y `ReportesEjecutivoPdfTemplate` hacia variables configurables.
- [ ] **Módulo de Licenciamiento:** Control de vigencia de servicio y bloqueo preventivo por vencimiento de suscripción.

---
*© 2026 BetoGraf_Inc SpA — Innovación, Arquitectura de Software y Servicios Tecnológicos.*
