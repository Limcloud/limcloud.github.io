---
title: "VKS Desde Cero – Parte 2: Arquitectura de VKS, entendiendo el Supervisor Cluster"
date: 2025-12-30 13:00:00 -0600
categories: [VMware, Kubernetes]
tags: [VKS, Supervisor, Kubernetes Architecture, vSphere, VCF]
pin: false
toc: true
comments: true

image:
  path: /assets/img/vks/04-supervisor-architecture-overview.png
  alt: "Arquitectura del Supervisor Cluster en vSphere Kubernetes Services"
---
En el post anterior vimos qué es vSphere Kubernetes Services (VKS) y qué habilita dentro de vSphere.  
Ahora toca entrar en la parte técnica y entender cómo está construida la arquitectura que hace posible esa integración.

El punto central de todo es el Supervisor Cluster. Entender qué es y qué rol cumple permite que el resto de conceptos —authorization, networking y storage— encajen de forma natural, especialmente para quienes ya administran entornos vSphere.

---

## Arquitectura de VKS a alto nivel

Desde el punto de vista de diseño, VKS parte de una idea clara: Kubernetes no se ejecuta como una plataforma externa, sino como una capacidad integrada al plano de control de vSphere.

Esto implica que vSphere sigue siendo responsable de la infraestructura, mientras Kubernetes se ejecuta sobre ella sin operar como un dominio aislado. vCenter mantiene visibilidad y control del entorno, y no existe una segunda plataforma que deba administrarse en paralelo.

A nivel conceptual, el stack se puede leer de arriba hacia abajo de la siguiente forma:

- Kubernetes control plane, expuesto a través del Supervisor  
- Integración directa con vCenter  
- ESXi como data plane  
- Networking y storage provistos por vSphere  

Esta integración es la base que permite gobernar Kubernetes con criterios enterprise sin alterar su funcionamiento nativo.

---

## El Supervisor Cluster

El Supervisor Cluster es un clúster Kubernetes especializado que se despliega y administra directamente desde vSphere.

No es un Kubernetes genérico instalado por el usuario ni un clúster pensado para ejecutarse de forma standalone. Es un Kubernetes diseñado específicamente para integrarse con vSphere y actuar como el punto de entrada para consumir Kubernetes dentro del entorno.

Desde una perspectiva práctica, el Supervisor es el control plane que vSphere utiliza para ofrecer capacidades Kubernetes de forma nativa. Su lifecycle, disponibilidad y operación están alineados con los mecanismos que ya existen en la plataforma.

---

## Componentes principales del Supervisor

Internamente, el Supervisor está compuesto por los elementos típicos de un Kubernetes control plane, desplegados como virtual machines y gestionados por vSphere.

Incluye componentes como el Kubernetes API Server, etcd y los control plane nodes, junto con servicios de integración que permiten a vSphere interactuar directamente con Kubernetes.

En el data plane, ESXi sigue siendo el responsable de ejecutar los workloads. Kubernetes no interactúa directamente con el hardware; esa abstracción sigue siendo responsabilidad de vSphere. Este punto es clave para entender por qué el Supervisor no reemplaza los conceptos de Kubernetes, sino que redefine cómo se conectan con la infraestructura subyacente.

---

## Supervisor Authorization (visión técnica para administradores vSphere)

Desde el punto de vista de un administrador vSphere, el modelo de authorization del Supervisor resulta bastante natural, aunque se exprese mediante constructos propios de Kubernetes.

El Supervisor se apoya directamente en los identity providers configurados en vCenter y en el modelo de permisos ya conocido. Usuarios y grupos se asignan a vSphere Namespaces utilizando roles definidos en vCenter, y estos roles se traducen internamente a Kubernetes RBAC.

En la práctica, esto permite que el acceso al Kubernetes API esté alineado con los controles corporativos existentes, evitando la proliferación de credenciales y configuraciones de RBAC difíciles de gobernar en Kubernetes standalone.

---

## Supervisor Networking (desde el diseño de red enterprise)

El networking del Supervisor mantiene los mismos principios que ya se aplican en entornos vSphere tradicionales: arquitectura definida, segmentación, aislamiento y control del tráfico.

Kubernetes consume conectividad dentro del modelo definido por infraestructura, sin operar como un dominio independiente. Esto permite que los equipos de infraestructura sigan aplicando prácticas conocidas, mientras que los equipos de plataforma y desarrollo trabajan con abstracciones propias de Kubernetes.

Más adelante entraremos en detalle en topologías, integración con NSX y exposición de servicios, pero el punto clave es que el Supervisor extiende el diseño de red vSphere al mundo cloud native sin fragmentar la operación.

---

## Supervisor Storage (del datastore al Persistent Volume)

El enfoque de storage del Supervisor muestra claramente la convergencia entre vSphere y Kubernetes.

Las storage policies siguen siendo el mecanismo principal de control desde infraestructura, mientras que Kubernetes consume storage mediante Persistent Volumes y Storage Classes respaldados directamente por esas políticas.

Para un administrador vSphere, esto implica que el modelo de storage no cambia radicalmente. Kubernetes consume storage de forma declarativa, pero siempre dentro de límites claros y gobernados, evitando escenarios comunes de descontrol en entornos Kubernetes standalone.

---

## Por qué el Supervisor es el punto clave

Authorization, networking y storage son los puntos donde tradicionalmente se rompe la relación entre infraestructura y plataformas modernas.

El Supervisor no intenta ocultar Kubernetes ni simplificarlo artificialmente. Ofrece un punto de integración donde las prácticas maduras de vSphere se trasladan al mundo de cargas cloud native sin fricción innecesaria.

---

## Qué sigue

A partir de aquí, el siguiente paso es profundizar cada uno de estos temas por separado:

- Arquitectura interna del Supervisor Cluster  
- Authorization y RBAC en detalle  
- Supervisor Networking  
- Supervisor Storage  

En el próximo post vamos a entrar en detalle en:

Supervisor Cluster: arquitectura interna y componentes
