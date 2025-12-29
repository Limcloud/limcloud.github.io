---
title: "VKS (vSphere Kubernetes Services): qué es y qué habilita en vSphere"
date: 2025-12-29 13:00:00 -0600
categories: [VMware, Kubernetes]
tags: [VKS, vSphere, Kubernetes, Supervisor, VCF]
pin: false
toc: true
comments: true
---

Si vienes de un mundo principalmente vSphere, es muy probable que ya hayas escuchado hablar de Kubernetes, contenedores, clusters, namespaces y todo el ecosistema cloud native.  
También es muy probable que hayas pensado algo como:

> *“Ok, suena interesante, pero ¿cómo encaja realmente Kubernetes dentro de mi infraestructura vSphere?”*

Ahí es donde entra **vSphere Kubernetes Services (VKS)**.

En este post voy a explicar **qué es VKS**, **qué problema resuelve** y **qué capacidades habilita dentro de vSphere**, sin entrar todavía en configuraciones ni pasos técnicos. La idea es construir una base clara antes de meternos de lleno en arquitectura y operación.

---

## ¿Qué es vSphere Kubernetes Services (VKS)?

VKS es el conjunto de capacidades dentro de vSphere que permiten **ejecutar y operar Kubernetes de forma nativa sobre vSphere**, utilizando los mismos componentes que ya existen en la plataforma: **vCenter, ESXi, networking y storage**.

No es un producto separado que se instala “encima” de vSphere.  
Tampoco es simplemente “Kubernetes corriendo en VMs”.

VKS es la **integración directa de Kubernetes en el core de vSphere**, donde vSphere pasa de ser solo una plataforma para virtual machines a convertirse también en una plataforma para **Kubernetes workloads**, bajo un mismo plano de control.

---

## El problema que VKS viene a resolver

Antes de VKS, el patrón típico era este:

- vSphere administra la infraestructura (compute, storage, networking)
- Kubernetes vive en otro lado:
  - En VMs gestionadas por otro equipo
  - O en una plataforma completamente distinta
- Doble operación:
  - Infra team administra vSphere
  - Otro equipo administra Kubernetes
- Poco alineamiento entre ambos mundos

Esto genera fricción real en entornos enterprise:
- Duplicación de herramientas
- Falta de visibilidad end-to-end
- Problemas de ownership
- Dificultad para escalar Kubernetes de forma controlada

VKS aparece para **romper ese silo** y unificar la operación.

![Before vs After: vSphere + Kubernetes vs vSphere + VKS](/assets/img/2025-01-15-vks-que-es-y-que-habilita/01-vks-before-after.png){: w="1200" h="675" }


---

## Qué habilita VKS dentro de vSphere

Cuando VKS está habilitado, vSphere deja de ser solo “la capa de infraestructura” y pasa a **entender Kubernetes como un ciudadano de primera clase**.

A nivel práctico, VKS habilita:

- Kubernetes integrado directamente con vCenter
- Gestión del lifecycle de Kubernetes desde la plataforma
- Uso nativo de:
  - Networking
  - Storage
  - Identity
- Un modelo claro de consumo basado en **Namespaces**
- Separación de responsabilidades entre infraestructura y plataformas

### Un detalle técnico importante

Cuando decimos que vSphere “entiende” Kubernetes, **no significa que vSphere reimplemente Kubernetes**.  
Significa que **vSphere se integra directamente con el Kubernetes control plane**, permitiendo que conceptos como networking, storage, identity y lifecycle se gestionen desde la plataforma, **sin abstraer ni esconder Kubernetes**.

Kubernetes sigue siendo Kubernetes.  
La diferencia está en **cómo se integra y gobierna**.

![Dónde vive VKS: capas en vSphere](/assets/img/2025-01-15-vks-que-es-y-que-habilita/02-vks-layers.png){: w="1200" h="675" }

---

## Qué NO es VKS

Para evitar confusiones comunes:

- VKS **no reemplaza Kubernetes**
- VKS **no elimina la necesidad de entender Kubernetes**
- VKS **no convierte Kubernetes en algo automático o trivial**

Desde el punto de vista operativo:
- `kubectl` sigue existiendo
- Los conceptos de Kubernetes no desaparecen
- Los equipos siguen trabajando con YAML y APIs

VKS **no simplifica Kubernetes**, pero **sí simplifica su integración en entornos vSphere enterprise**.

{: .prompt-tip }
**Tip rápido:** si tu objetivo es “no aprender Kubernetes”, VKS no es el camino. Si tu objetivo es **operarlo bien en vSphere**, ahí sí.

---

## VKS vs Kubernetes “standalone”

Una forma simple de verlo:

| Kubernetes standalone | Kubernetes con VKS |
|---|---|
| Vive fuera de vSphere | Vive dentro de vSphere |
| Infra y K8s separados | Infra y K8s integrados |
| Gestión fragmentada | Gestión centralizada |
| Difícil de gobernar | Gobernable a escala |

Esta diferencia es clave cuando hablamos de **seguridad, control y operación** a largo plazo.

---

## ¿Para quién tiene sentido VKS?

VKS tiene muchísimo sentido si:

- Ya usas vSphere como plataforma estándar
- Quieres habilitar Kubernetes sin crear otra “isla tecnológica”
- Necesitas:
  - Multi-tenancy
  - Quotas
  - RBAC
  - Integración con infraestructura existente
- Buscas un modelo operativo más claro entre:
  - Infra team
  - Platform team
  - Development teams

No es solo una decisión técnica, es una decisión **operativa y organizacional**.

---

## Idea clave para cerrar

> **VKS no es “Kubernetes sobre VMs”.  
> Es Kubernetes integrado al plano de control de vSphere.**

VKS no elimina la complejidad de Kubernetes, pero **sí elimina la complejidad de integrarlo y gobernarlo correctamente en entornos vSphere enterprise**.

![Puente al Post 2: Control Plane vs Data Plane](/assets/img/2025-01-15-vks-que-es-y-que-habilita/03-control-plane-data-plane.png){: w="1200" h="675" }

---

## Qué sigue

En el próximo post vamos a subir un nivel técnico y responder una pregunta clave:

👉 **¿Cómo corre Kubernetes sobre vSphere a nivel de arquitectura?**

Ahí empezamos a hablar de **control plane, data plane, ESXi y la integración real bajo el capó**.
