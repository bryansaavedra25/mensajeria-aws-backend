# 🚀 Backend: Sistema de Gestión de Órdenes y Alertas (Spring Boot + AWS SQS/SNS)

Este repositorio contiene la API REST y el motor de procesamiento asíncrono para el sistema de gestión de órdenes. Implementa una arquitectura orientada a eventos (*Event-Driven Architecture*) utilizando servicios de **Amazon Web Services (AWS)** para garantizar el procesamiento de colas, tolerancia a fallos mediante Dead Letter Queue (DLQ) y monitoreo automatizado.

---

## 🛠️ Tecnologías Utilizadas

- **Lenguaje:** Java 17
- **Framework:** Spring Boot 3.3.4
- **Módulo AWS:** Spring Cloud AWS / AWS SDK v2
- **Servicios Cloud (AWS):**
  - **AWS SQS (Simple Queue Service):** Cola principal (`ordenes-q`) y Dead Letter Queue (`ordenes-dlq`).
  - **AWS SNS (Simple Notification Service):** Tema Estándar (`alertas-dlq-topic`) para envío de notificaciones.
  - **AWS CloudWatch:** Alarma de monitoreo sobre la métrica de la DLQ.
- **Gestor de Dependencias:** Maven

---

## ⚡ Reglas de Negocio y Manejo de Errores

1. **Flujo Exitoso:**
   - Mensajes con un monto positivo (`monto > 0`) se procesan correctamente y se confirman (*ACK*) en SQS.

2. **Falla Transitoria (Reintentos SQS):**
   - Si el nombre del cliente es `"Cliente 1"`, la aplicación simula un error temporal lanzando una excepción. SQS reintenta el mensaje según la política de redireccionamiento configurada.

3. **Falla Definitiva (DLQ Directa):**
   - Si la orden contiene un monto inválido o negativo (`monto < 0`), el sistema la clasifica como una falla no recuperable y la envía a la **Dead Letter Queue (`ordenes-dlq`)**.

4. **Alertamiento Automatizado:**
   - Al acumularse mensajes en la DLQ, **CloudWatch** dispara una alarma que notifica vía **AWS SNS** por correo electrónico a la dirección registrada.

---

## 📡 Endpoints de la API

| Método | Endpoint | Descripción | Cuerpo de la Petición (JSON) |
| :--- | :--- | :--- | :--- |
| `POST` | `/ordenes` | Envío asíncrono de orden a la cola principal SQS | `{"id": 201, "cliente": "Cliente Test", "monto": 500}` |
| `POST` | `/ordenes/sync` | Procesamiento síncrono inmediato | `{"id": 201, "cliente": "Cliente Test", "monto": 500}` |
| `GET` | `/estado` | Consulta el número de mensajes en las colas (Principal y DLQ) | N/A |

---

## ⚙️ Configuración y Requisitos

### Prerrequisitos
- Java 17 o superior instalado.
- Credenciales de AWS configuradas en el entorno local (`~/.aws/credentials` o variables de entorno) con acceso a SQS, SNS y CloudWatch.

### Variables de Configuración (`application.properties`)
-properties
-server.port=8080

# Configuración AWS Region
-spring.cloud.aws.region.static=us-east-1

# Nombres de Colas y Temas
-aws.sqs.queue.main=ordenes-q
-aws.sqs.queue.dlq=ordenes-dlq
-aws.sns.topic.alertas=arn:aws:sns:us-east-1:ACCOUNT_ID:alertas-dlq-topic

## 🚀 Ejecución en Local
Clonar el repositorio:
-git clone https://github.com/bryansaavedra25/mensajeria-aws-backend.git
-cd mensajeria-aws-backend
-Compilar y ejecutar la aplicación:
-./mvnw spring-boot:run
-El servidor se iniciará en http://localhost:8080.

👤 Autor
Nombre: Bryan Saavedra
        Elena Espinoza
Sección: 001D



