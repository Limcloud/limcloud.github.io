---
title: "Habilitando vSphere Supervisor utilizando NSX VPCº"
date: 2026-03-24 09:00:00 -0600
categories: [VMware, Kubernetes]
tags: [VCF, vSphere Supervisor, NSX, VPC, VKS, Kubernetes]
toc: true
comments: true
---

## Introducción

En este artículo voy a mostrar cómo habilitar vSphere Supervisor utilizando VCF Networking with VPC, que es el modelo más moderno para este tipo de despliegues dentro de VMware Cloud Foundation.

La idea no es solamente recorrer el asistente, sino también entender un poco mejor qué estamos configurando y por qué ciertos campos son importantes, especialmente cuando entran en juego conceptos como Project, VPC Profile, Service CIDR y la conectividad hacia el Tier-0 Gateway.

---

## Antes de empezar

Hay algunos conceptos que es importante tener claros desde el inicio:

- **vSphere Zone**: representa un clúster de vSphere donde puede vivir el Supervisor.
- **Supervisor**: es el plano de control de Kubernetes integrado directamente en vSphere.
- **Project**: límite lógico de consumo y aislamiento para networking y workloads.
- **VPC Profile**: define cómo se crean y consumen redes dentro del modelo VPC en NSX.
- **VPC (Virtual Private Cloud)**: entorno de red aislado dentro de NSX que permite desplegar workloads (Pods y VMs) con su propio espacio de direcciones IP, routing y políticas, similar a lo que se ve en nubes públicas.
- **Service CIDR**: rango interno de direcciones IP usado por Kubernetes para servicios tipo ClusterIP y otros componentes internos.
- **Tier-0 Gateway**: salida north-south de las redes creadas dentro del modelo de VPC.

---

## Paso 1 – Entrar a Supervisor Management

En **vCenter**, abrimos el menú principal y seleccionamos **Supervisor Management**.

![Supervisor Management](./assets/img/2026-03-24-habilitar-supervisor-nsx-vpc/01.png)

Luego hacemos clic en **Get Started**.

![Get Started](./assets/img/2026-03-24-habilitar-supervisor-nsx-vpc/02.png)

---

## Paso 2 – Seleccionar el tipo de networking

En el asistente, seleccionamos la opción:

**VCF Networking with VPC**

![VCF Networking with VPC](./assets/img/2026-03-24-habilitar-supervisor-nsx-vpc/03.png)

Este punto es importante porque cambia completamente la lógica del despliegue. En lugar de trabajar con segmentos tradicionales, el Supervisor se integra con el modelo de NSX VPC, donde el aislamiento y consumo de redes gira alrededor de Projects y VPCs.

---

## Paso 3 – Seleccionar la vSphere Zone y definir el nombre del Supervisor

Antes de llegar a este punto, ya debemos haber creado al menos un vSphere Zone.

![vSphere Zone](./assets/img/2026-03-24-habilitar-supervisor-nsx-vpc/04.png)

Aquí definimos en qué Zone se va a desplegar el Supervisor y además le asignamos un nombre.

Un detalle importante es que:

- Si usamos un solo vSphere Zone, estamos hablando de un despliegue sencillo, útil para laboratorio o ambientes pequeños.
- Si queremos una arquitectura con mayor resiliencia, podemos usar tres Zones, lo que permite distribuir el control plane entre distintos clusters.

En mi caso utilicé una sola vSphere Zone porque el objetivo era un laboratorio.

---

## Paso 4 – Habilitar alta disponibilidad del control plane

En esta misma etapa aparece la opción **Enable control plane high availability**.

![Control Plane HA](./assets/img/2026-03-24-habilitar-supervisor-nsx-vpc/05.png)

Al marcarla, el despliegue crea tres VMs para el control plane del Supervisor.

Se puede desplegar un único nodo, pero esto únicamente se recomienda para lab o pruebas. Para producción, tres nodos es la decisión correcta.

---

## Paso 5 – Seleccionar la política de almacenamiento

Después configuramos el almacenamiento que va a consumir el Supervisor.

![Storage Policy](./assets/img/2026-03-24-habilitar-supervisor-nsx-vpc/06.png)

Aquí debemos seleccionar un Storage Policy compatible con el datastore que queremos usar. En mi ejemplo utilicé vSAN, pero puede ser cualquier policy válido para el entorno.

---

## Paso 6 – Configurar la red de management

En la sección de management network definimos la red donde van a desplegarse los nodos del control plane.

![Management Network](./assets/img/2026-03-24-habilitar-supervisor-nsx-vpc/07.png)

El Supervisor requiere un bloque de al menos 5 direcciones IP consecutivas.

Este rango es para cubrir:

- IPs de los nodos del control plane  
- IP del endpoint de Kubernetes  
- Direcciones reservadas para el funcionamiento interno  

Además:

- Deben ser IPs estáticas  
- DNS debe estar bien configurado (Forward and reverse) 
- debe existir conectividad hacia vCenter y NSX  

---

## Paso 7 – Configurar Workload Networking con NSX VPC

Aquí es donde se debe configurar el modelo de VPC.

![Workload Networking](./assets/img/2026-03-24-habilitar-supervisor-nsx-vpc/08.png)

Seleccionamos:

- **Project**
- **VPC Profile**

Aunque desde el wizard se ve simple, detrás de eso están pasando varias cosas importantes:

- El Supervisor se apoya en el **Project** para el aislamiento lógico.
- El **VPC Profile** define cómo se consumen los rangos de red.
- Se crea una **VPC** donde vivirán los workloads.

---

## Paso 7.1 – Elegir correctamente el Service CIDR

Elegir correctamente el Service CIDR es clave ya que se usa internamente por Kubernetes para:

- Servicios tipo ClusterIP  
- Componentes internos  
- Servicios de plataforma  


El Service CIDR no debe hacer overlap con:

- redes físicas  
- redes de management  
- redes NSX  
- otros bloques existentes  

Ejemplo común:

```
10.96.0.0/16
```

## Paso 8 – Seleccionar el tamaño del Control Plane

![Control Plane Size](./assets/img/2026-03-24-habilitar-supervisor-nsx-vpc/09.png)

Aquí definimos el tamaño de las VMs del control plane.

En este caso seleccioné Small, ya que es un laboratorio.

Este parámetro impacta directamente:

- recursos disponibles del Supervisor
- cantidad de namespaces soportados
- capacidad general del cluster

En esta misma pantalla también podemos marcar **Export Configuration**, lo cual genera un JSON con toda la configuración.

## Paso 9 – Revisar y desplegar

![Review](./assets/img/2026-03-24-habilitar-supervisor-nsx-vpc/10.png)

Aquí revisamos toda la configuración antes de continuar:

- Zone
- Storage
- Networking
- VPC settings
- Control plane size

Si todo está correcto, damos clic en Finish.

## Verificación

Una vez finalizado el despliegue, podemos validar el estado desde Supervisor Management.



Los estados deben aparecer como:

- Config Status → Running
- Running Status → Running

---

## ¿Qué se despliega junto con el Supervisor?

Una vez habilitado el Supervisor, ya quedan disponibles varios componentes base del stack.

![Review](./assets/img/2026-03-24-habilitar-supervisor-nsx-vpc/11.png)

Entre los principales:

- Control plane de Kubernetes integrado en vSphere  
- VM Service  
- Capacidades para habilitar servicios adicionales como VKS, Harbor, entre otros  

A partir de este punto ya puedes empezar a consumir namespaces, desplegar workloads y habilitar servicios sobre el Supervisor.

---

## Consideraciones de networking

En el modelo de VCF Networking with VPC, las redes del Supervisor no son aisladas de forma independiente, sino que forman parte del diseño general de NSX.

El tráfico hacia afuera (north-south) se maneja a través del Tier-0 Gateway, por lo que es importante considerar desde el diseño:

- Cómo se van a anunciar las rutas (BGP o estático)  
- Conectividad hacia redes externas  
- Evitar overlap de direccionamiento  
