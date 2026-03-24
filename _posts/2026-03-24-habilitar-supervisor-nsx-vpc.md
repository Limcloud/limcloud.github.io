---
title: "Habilitando vSphere Supervisor utilizando NSX VPC"
date: 2026-03-24 13:00:00 -0600
categories: [VMware, Kubernetes]
tags: [VKS, Supervisor, Kubernetes Architecture, vSphere, VCF]
pin: false
toc: true
comments: true

---
En este artículo voy a mostrar cómo habilitar vSphere Supervisor utilizando VCF Networking with VPC, que es el modelo más moderno para este tipo de despliegues dentro de VMware Cloud Foundation.

La idea no es solamente recorrer el asistente, sino también entender un poco mejor qué estamos configurando y por qué ciertos campos son importantes, especialmente cuando entran en juego conceptos como Project, VPC Profile, Service CIDR y la conectividad hacia el Tier-0 Gateway.

Hay algunos conceptos que es importante tener claros desde el inicio:

- **vSphere Zone**: representa un clúster de vSphere donde puede vivir el Supervisor.
- **Supervisor**: es el plano de control de Kubernetes integrado directamente en vSphere.
- **Project**: límite lógico de consumo y aislamiento para networking y workloads.
- **VPC Profile**: define cómo se crean y consumen redes dentro del modelo VPC en NSX.
- **VPC (Virtual Private Cloud)**: entorno de red aislado dentro de NSX que permite desplegar workloads (Pods y VMs) con su propio espacio de direcciones IP, routing y políticas, similar a lo que se ve en nubes públicas.
- **Service CIDR**: rango interno de direcciones IP usado por Kubernetes para servicios tipo ClusterIP y otros componentes internos.
- **Tier-0 Gateway**: salida north-south de las redes creadas dentro del modelo de VPC.
