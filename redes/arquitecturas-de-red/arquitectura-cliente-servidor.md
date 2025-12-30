# Arquitectura Cliente Servidor

La arquitectura cliente-servidor es un modelo de diseño fundamental en redes que divide las tareas entre dos tipos de participantes: los clientes y los servidores.

* El Cliente solicita servicios.
* El Servidor proporciona esos servicios.

#### Componentes Principales

1. **El cliente (Inicia la comunicación y solicita servicios)**
2. **La red (Permite la comunicación entre el cliente y el servidor)**
3. **El servidor (Espera pasivamente las peticiones de los clientes, las procesa y envia una respuesta o el servicio esperado)**

#### **El proceso es un ciclo simple de solicitud-respuesta:**

1. **Solicitud (Request)**: El cliente (tu navegador) envía una solicitud a través de la red al servidor. Por ejemplo, "Quiero la página <kbd>nasa.gov</kbd> ".
2. **Procesamiento:** El servidor (el servidor de la nasa) recibe la solicitud. Busca la página, procesa la información y prepara una respuesta.
3. **Respuesta (Response)**: El servidor envía la respuesta (los datos de la página web) de vuelta al cliente a través de la red.
4. **Presentación**: El cliente (tu navegador) recibe los datos y te los muestra de forma entendible (renderiza la página web).

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>
