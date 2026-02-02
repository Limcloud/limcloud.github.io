---
title: "VKS Desde Cero – Parte 2: Arquitectura de VKS, entendiendo el Supervisor Cluster"
date: 2026-02-01 13:00:00 -0600
categories: [VMware, Kubernetes]
tags: [VKS, Supervisor, Kubernetes Architecture, vSphere, VCF]
pin: false
toc: true
comments: true

---

En el post anterior vimos qué es **vSphere Kubernetes Services (VKS)** y qué habilita dentro de vSphere.  
Ahora toca entrar en la parte técnica y entender **la arquitectura que hace posible todo esto**.

El punto central de VKS es el **Supervisor Cluster**.  
Si entiendes qué es, cómo se despliega y qué componentes lo conforman, conceptos como authorization, networking y storage empiezan a encajar de forma natural, especialmente si ya administras entornos vSphere.

---

## Arquitectura de VKS a alto nivel

Desde el punto de vista de diseño, VKS parte de una idea muy clara:

Kubernetes no se ejecuta como una plataforma externa, sino como una **capacidad integrada al plano de control de vSphere**.

Esto significa que:

- vSphere sigue siendo responsable de la infraestructura
- Kubernetes no opera como un dominio aislado
- vCenter mantiene visibilidad y control
- No existe una “segunda plataforma” que administrar en paralelo

A nivel conceptual, el stack se puede leer de arriba hacia abajo así:

- Kubernetes control plane, expuesto a través del Supervisor  
- Integración directa con vCenter  
- ESXi como data plane  
- Networking y storage provistos por vSphere  

Esta integración es la base que permite gobernar Kubernetes con criterios enterprise sin alterar su funcionamiento nativo.

---

## ¿Qué es realmente el Supervisor Cluster?

El **Supervisor Cluster** es un clúster Kubernetes especializado que se despliega y administra directamente desde vSphere.

No es un Kubernetes genérico instalado por el usuario ni un clúster standalone.  
Es un Kubernetes diseñado específicamente para integrarse con vSphere y actuar como el **punto de entrada para consumir Kubernetes dentro del entorno**.

Desde una perspectiva práctica:

- El Supervisor es el control plane que habilita Kubernetes en vSphere
- Su lifecycle lo gestiona vSphere
- Su disponibilidad depende de los mecanismos ya conocidos (HA, DRS, etc.)

---

## Componentes principales del Supervisor

### Control Plane del Supervisor

El Supervisor puede desplegarse con:

- 1 control plane VM (sin alta disponibilidad)
- 3 control plane VMs (recomendado)

En un despliegue con HA:

- Cada VM tiene su propia IP
- Existe una IP flotante para acceso al API
- Se reserva una IP adicional para procesos de parcheo
- vSphere DRS decide la ubicación de las VMs y las migra si es necesario

Cuando el Supervisor se despliega sobre **tres vSphere Zones**, cada control plane VM vive en una zona distinta, evitando puntos únicos de falla.

---

### VKS y Cluster API

Dentro del Supervisor corren los módulos responsables de:

- Aprovisionar clusters VKS
- Escalar nodos
- Gestionar el lifecycle del cluster

Todo esto se apoya en **Cluster API**, pero profundamente integrado con vSphere, sin perder compatibilidad con los conceptos estándar de Kubernetes.

---

### Virtual Machine Service

El **VM Service** permite que Kubernetes consuma máquinas virtuales como recursos declarativos.

Gracias a este servicio, el Supervisor puede:

- Desplegar VMs standalone
- Desplegar las VMs que conforman un cluster VKS

Para vSphere, siguen siendo VMs.  
Para Kubernetes, son recursos gestionables vía API.

---

### Spherelet

Cada host ESXi ejecuta un proceso llamado **Spherelet**.

Spherelet es, conceptualmente, un kubelet portado de forma nativa a ESXi.  
No corre dentro de una VM y permite que el host ESXi forme parte directa del cluster Kubernetes del Supervisor.

Este componente es clave para entender por qué el Supervisor elimina capas intermedias que normalmente existen en Kubernetes tradicional.

---

### Container Runtime Executive (CRX)

Los **vSphere Pods** se ejecutan sobre el componente **CRX**.

Desde la perspectiva de vCenter:

- CRX se comporta como una VM
- Usa virtualización por hardware
- Tiene aislamiento a nivel de hipervisor

Pero internamente:

- Utiliza un kernel Linux paravirtualizado
- Implementa direct boot
- Omite pasos innecesarios del arranque tradicional

El resultado es que los vSphere Pods arrancan casi tan rápido como contenedores, manteniendo el aislamiento esperado en entornos enterprise.

---

## vSphere Namespaces: el punto de consumo

Una vez creado el Supervisor, los recursos se consumen a través de **vSphere Namespaces**.

Desde infraestructura:

- Se definen cuotas
- Se asignan vSphere Zones
- Se controla storage y permisos

Desde DevOps:

- Se despliegan vSphere Pods
- Se crean clusters VKS
- Se consumen VMs vía VM Service

Un Namespace puede contener una combinación de todos estos recursos, siempre bajo control de vSphere.

---

## Aislamiento entre Management y Workloads

El Supervisor permite separar claramente:

- Plano de management
- Plano de workloads

Esto se logra mediante **vSphere Zones**, donde cada zona mapea a un cluster vSphere y puede tratarse como un dominio de falla independiente.

Un vSphere Namespace:

- Requiere al menos una vSphere Zone
- Puede usar hasta tres vSphere Zones

En un despliegue de tres zonas, los recursos se distribuyen de forma equitativa entre los clusters subyacentes, proporcionando resiliencia y balance.

---

## Opciones de despliegue del Supervisor

### Supervisor de una sola zona

- Un solo cluster vSphere
- HA a nivel de cluster
- Arquitectura simple
- Ideal para labs o entornos pequeños

---

### Supervisor de tres zonas

- Control plane altamente disponible
- Cada control plane VM en una zona distinta
- Protección ante fallas de un cluster completo

En este escenario, **la alta disponibilidad del control plane es obligatoria**.

---

## El Supervisor como punto de convergencia

Authorization, networking y storage suelen ser los puntos donde Kubernetes y la infraestructura tradicional entran en conflicto.

El Supervisor no intenta ocultar Kubernetes ni simplificarlo artificialmente.  
Actúa como una capa de integración donde las prácticas maduras de vSphere se trasladan al mundo cloud native sin fricción innecesaria.

---

## Qué sigue

A partir de aquí vamos a profundizar cada tema por separado:

- Arquitectura interna del Supervisor Cluster  
- Authorization y RBAC en detalle  
- Supervisor Networking  
- Supervisor Storage  

En el próximo post entramos de lleno en:

**Supervisor Cluster: arquitectura interna y componentes**
