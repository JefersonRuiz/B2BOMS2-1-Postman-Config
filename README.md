# CONFIGURACIÓN POSTMAN

En este directorio se entregan dos archivos, uno correspondiente a la plantilla o *template* para el entorno en Postman (<code>SF Environment Template.postman_environment.json</code>), donde se almacenarán los datos de configuración para la conexión con Salesforce, y otro con la colección de los diferentes llamados o solicitudes que se realizan para validar el correcto funcionamiento y configuración de las pruebas.

## Pre-requisitos

Para poder conectarse correctamente por medio de las APIs de Salesforce, sean estas aquellas ya provistas por Salesforce, o creadas especificamente para un entorno determinado, es necesario contar con lo siguiente:

* Un usuario activo en el entorno de Salesforce, con permisos suficientes para poder hacer llamados hacia Salesforce.

* Una aplicación conectada configurada con los permisos necesarios, y que el usuario tenga a su asignado el uso de dicha aplicación conectada.

Para la integración se está utilizando inicialmente la aplicación conectada **AccountResAPI**.


## Plantilla de entorno

La plantilla de entorno se establecen las variables de entorno necesarias que las solicitudes funcionen correctamente, y de esta manera poder autorizar y conectar con entornos de Salesforce. Se recomienda duplicar el template entregado para la conexión con un nuevo entorno de Salesforce, dandole un nombre adecuada que identifique facilmente a qué entorno de Salesforce corresponde. Dichas variables son las siguientes:

* **Variables asociadas a la aplicación conectada (*Connected App*)**:
    * <code>sf_client_id</code>: *Hash* correspondiente al identificador de la aplicación conectada.
    * <code>sf_client_secret</code>: *Hash* correspondiente al secreto asociado a la aplicación conectada.

    Estos valores pueden ser los mismos para todos los usuarios de una misma organización, pero serán diferentes para cada una de las organizaciones de Salesforce, aún si se usa la misma aplicación conectada.

* **Variables asociadas al usuario de Salesforce**:
    * <code>sf_user_name</code>: Usuario del Salesforce.
    * <code>sf_password</code>: Contraseña del usuario de Salesforce.
    * <code>sf_user_token</code>: Token de seguridad asociado al usuario de Salesforce.

    Normalmente el usuario y su respectiva contraseña se entregan o configuran durante la creación del usuario. El token de seguridad será necesario generarlo la primera vez dentro de la configuración del usuario dentro de la plataforma de Salesforce.

* **Variables Asociadas al entorno de Salesforce**
    * <code>instance_url</code>: Corresponde a que tipo de organización se está tratando de acceder. Esta variable sera igual a <code>https://login.salesforce.com</code> si el entorno es uno productivo o *developer edition*, o será <code>https://test.salesforce.com</code> si el entorno es un *sandbox*.
    * <code>sf_url</code>: URL de la versión *classic* del entorno de Salesforce (https://\<<my-instance\>>.my.salesforce.com).
    * <code>access_token</code>: Token de autorización generado al realizar el llamado de autenticacíon y autorización (más adelante se ve este proceso).
    * <code>version</code>: Versión de API de Salesforce usada.

Si se necesita, siempre es posible agregar más variables de acuerdo a las necesidades y preferencias de cada usuario.

## Colección de llamados.

### [POST] Get Auth Token

Este es el llamado que hay que realizar inicialmente para obtener el token de acceso a Salesforce. Si la configuración del entorno explicada previamente se hizo correctamente, y el usuario y la aplicación conectada tienen los permisos y configuraciones necesarias, sólo hará falta dar clic al botón *"Send"* y se obtendrá el token de acceso.

El llamado es de tipo *POST*, y su cuerpo está construido como se muestra a continuación:

| Key   |      Value      |
|----------|:-------------:|
| grant_type | `"password"` |
| client_id |    `{{sf_client_id}}`   |
| client_secret | `{{sf_client_secret}}` |
| username | `{{sf_user_name}}` |
| password | `{{sf_password}}{{sf_user_token}}` |

donde los valores entre corchetes son determinados dinamicamente de acuerdo a lo establecido en el entorno, como ya se explicó previamente.

Si el llamado se ejecuta correctamente, la respuesta deberá ser similar a lo siguiente:

    {
        "access_token": "<<BIG HASH - YOUR TOKEN ID>>",
        "instance_url": "https://<<YOUR_INSTANCE>>.my.salesforce.com",
        "id": "https://login.salesforce.com/id/<<SOME_ID>>",
        "token_type": "Bearer",
        "issued_at": "<<1721588207274>>",
        "signature": "<<ANOTHER HASH>>="
    }

Como parte de la configuración del llamado, existe un `script` que se ejecuta después de llamado que es el siguiente:

    const jsonData = pm.response.json();
    const id = jsonData.id.split('/');

    if (!jsonData.error) {
        const context = pm.environment.name ? pm.environment : pm.collectionVariables;
        context.set("access_token", jsonData.access_token);
    }

Que basicamente establece la variable de entorno `access_token` con el valor de la variable `access_token` encontrada en el cuerpo de la respuesta.

### [POST] Actualizar inventario de productos

**REVISAR**

Este llamado actualizar el inventario de los productos para una locación determinada.

Antes de ejecutar este llamado es necesario primero realizar el llamado para obtener el token de autorización explicado anteriormente, ya que el método de autentincación y autorización requiere el token de acceso que se obtiene como ya se explicó previamente. Es necesario además configurar el cuerpo de la solicitud agregando el archivo `json` llamado `uploadWithFuture`. Este archivo tiene la siguiente estructura:

    {
        "recordId":"20240124128-690247-1240",
        "onHand":1,
        "sku":"000000000000102394",
        "effectiveDate":"2024-05-26T14:05:22.781-07:00",
        "safetyStockCount": 0,
        "locationId":"1226"
    }

donde el `sku` se corresponde con el mismo sku del producto en Salesforce, pero antecedido de 12 ceros. La clave `locationId` se corresponde con el código del área de venta donde se quiere agregar o modificar el inventario. el valor `onHand` se corresponde con el número de *stock* de las unidades correspondiente al producto en el aŕea de venta. Para que la prueba se ejecute correctamente es necesario que el área de venta donde se quiera modificar el inventario corresponda con el área de venta del solicitante o cliente. Más información ver el siguiente enlace:

https://drive.google.com/file/d/1HRIskZQ--xRCZDTrL6LU90WL8L6QK5ab/view



