# PopIn-PaymentFormT1-Vue

Esta página explica cómo crear un formulario de pago dinámico desde cero utilizando VUE.JS y vue-cli, con la biblioteca de embedded-form-glue.

<p align="center">
  <img src="/src/assets/imagen-popin.png?raw=true" alt="Formulario"/>
</p> 

<a name="Requisitos_Previos"></a>

## Requisitos Previos

* Extraer credenciales del Back Office Vendedor. [Guía Aquí](https://github.com/izipay-pe/obtener-credenciales-de-conexion)

* Debe instalar la [versión de LTS node.js](https://nodejs.org/es/).

## 1.- Crear el proyecto

* Descargar el proyecto .zip haciendo click [Aquí](https://github.com/izipay-pe/PopIn-PaymentFormT1-Vue/archive/refs/heads/main.zip) o clonarlo desde Git. 
 
  ```sh
  git clone https://github.com/izipay-pe/PopIn-PaymentFormT1-Vue.git
  ``` 

* Ingrese a la carpeta raiz del proyecto.

* A continuación, instale el cliente vue-cli:

  ```bash
  npm install -g @vue/cli
  ```

  Más detalles en la página web de [vue-cli web-site](https://cli.vuejs.org/guide/installation.html).

* Agregue la dependencia con:

  ```bash
  npm install --save @lyracom/embedded-form-glue
  ```

* Ejecútelo y pruébelo con el comando:

  ```sh
  npm run serve
  ```

  ver el resultado en http://localhost:8080/

## 2.- Agregar el formulario de pago

**Nota**: Reemplace **[CHANGE_ME]** con sus credenciales de `API REST` extraídas desde el Back Office Vendedor, ver [Requisitos Previos](#Requisitos_Previos).

* Editar en public/index.html en la sección HEAD.

  ```html
    <!-- tema y plugins. debe cargarse en la sección HEAD -->
    <link rel="stylesheet"
    href="~~CHANGE_ME_ENDPOINT~~/static/js/krypton-client/V4.0/ext/classic-reset.css">
    <script
        src="~~CHANGE_ME_ENDPOINT~~/static/js/krypton-client/V4.0/ext/classic.js">
    </script>
  ```
* Edite el componente predeterminado src/app/AddForm.vue ingresando sus credenciales extraidas desde [Requisitos Previos](#Requisitos_Previos). 

  ```javascript
    const endpoint = "~~CHANGE_ME_ENDPOINT~~";
    const publicKey = "~~CHANGE_ME_PUBLIC_KEY~~";
    const formToken = "DEMO-TOKEN-TO-BE-REPLACED";
  ```

  Puede generar un `formToken` de prueba ingresando a la pestaña `Pruébame` dentro del siguiente link: [Aquí](https://secure.micuentaweb.pe/doc/es-PE/rest/V4.0/api/playground/Charge/CreatePayment/).

  ![token](/src/assets/formToken.png)

* Actualice los estilos dentro de src/App.vue style:

  ```html
  <style scoped>
    .container {
      display: flex;
      flex-direction: row;
      justify-content: center;
      align-items: center;
    }
  </style>
  ```

  ## 2.1 Interactuar con un endpoint propio

    Edite el componente predeterminado src/app/AddForm.vue, con el siguiente codigo si quiere interactuar con el formulario de pago, con un endpoint propio.

    ```html
    <template>
      <div class="hello">
        <h1>{{ msg }}</h1>
        <div class="container">
          <div id="myPaymentForm">
            <div class="kr-embedded" kr-popin>
              <div class="kr-pan"></div>
              <div class="kr-expiry"></div>
              <div class="kr-security-code"></div>
              <div class="kr-form-error"></div>
              <button class="kr-payment-button"></button>
            </div>
          </div>
        </div>
        <div data-test="payment-message">{{ message }}</div>
      </div>
    </template>

    <script>
    /* import embedded-form-glue library */
    import KRGlue from '@lyracom/embedded-form-glue'
    import axios from 'axios'

    export default {
      name: 'AttachForm',
      props: {
        msg: String
      },
      data() {
        return {
          message: ''
        }
      },
      mounted() {
        const endpoint = '~~CHANGE_ME_ENDPOINT~~'
        const publicKey = '~~CHANGE_ME_PUBLIC_KEY~~'
        let formToken = 'DEMO-TOKEN-TO-BE-REPLACED'

        // Generate the form token
        axios
          .post('http://localhost:3000/createPayment', {
            paymentConf: { amount: 10000, currency: 'USD' }
          })
          .then(resp => {
            formToken = resp.data
            return KRGlue.loadLibrary(
              endpoint,
              publicKey
            ) /* Load the remote library */
          })
          .then(({ KR }) =>
            KR.setFormConfig({
              /* set the minimal configuration */
              formToken: formToken,
              'kr-language': 'en-US' /* to update initialization parameter */
            })
          )
          .then(({ KR }) => KR.onSubmit(this.validatePayment)) // Custom payment callback
          .then(({ KR }) =>
            KR.attachForm('#myPaymentForm')
          ) /* create a payment form */
          .then(({ KR, result }) => {
            KR.showForm(result.formId)
            this.ready = true
          }) /* show the payment form */
          .catch(
            error => (this.message = error + ' (see console for more details)')
          )
      },
      methods: {
        /* Validate the payment data */
        validatePayment(paymentData) {
          axios
            .post('http://localhost:3000/validatePayment', paymentData)
            .then(response => {
              if (response.status === 200) this.message = 'Payment successful!'
            })
          return false // Return false to prevent the redirection
        }
      }
    }
    </script>

    <!-- Add "scoped" attribute to limit CSS to this component only -->
    <style scoped>
    .container {
      display: flex;
      flex-direction: row;
      justify-content: center;
      align-items: center;
    }
    </style>
    ```

## 3.- Transacción de prueba

El formulario de pago está listo, puede intentar realizar una transacción utilizando una tarjeta de prueba con la barra de herramientas de depuración (en la parte inferior de la página).

  ![tarjeta](/src/assets/tarjetas-prueba.png)

Para obtener más información, eche un vistazo a:

- [Formulario incrustado: prueba rápida](https://secure.micuentaweb.pe/doc/es-PE/rest/V4.0/javascript/quick_start_js.html)
- [Primeros pasos: pago simple](https://secure.micuentaweb.pe/doc/es-PE/rest/V4.0/javascript/guide/start.html)
- [Servicios web - referencia de la API REST](https://secure.micuentaweb.pe/doc/es-PE/rest/V4.0/api/reference.html)

## 4.- Implementar IPN

* Ver manual de implementacion de la IPN [Aquí](https://secure.micuentaweb.pe/doc/es-PE/rest/V4.0/kb/payment_done.html).

* Ver el ejemplo de la respuesta IPN con PHP [Aquí](https://github.com/izipay-pe/Redirect-PaymentForm-IpnT1-PHP).

* Ver el ejemplo de la respuesta IPN con NODE.JS [Aquí](https://github.com/izipay-pe/Response-PaymentFormT1-Ipn).
