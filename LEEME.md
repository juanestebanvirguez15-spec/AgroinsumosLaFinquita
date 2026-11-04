# Agroinsumos LaFinquita: cómo publicar y activar los pagos

La página (`index.html`) funciona sola: puedes abrirla en el navegador para ver el catálogo, el carrito y el WhatsApp.
El **pago en línea** usa las funciones de la carpeta `api/`, que hablan con ePayco desde el servidor.
Las llaves nunca van dentro de la página.

## Modos de pago

| Modo | Cuándo se activa | ¿Mueve dinero? |
|---|---|---|
| **Simulación** | Solo al abrir la página en tu computador (archivo o `localhost`). En el dominio publicado no existe. | No. Nunca llama a ePayco ni al servidor. |
| **Pruebas ePayco** | Página publicada con `EPAYCO_TEST=true`. | No. Usa el entorno de pruebas de ePayco. |
| **Producción** | Página publicada con `EPAYCO_TEST=false`. | Sí. |

## 1. Datos de ePayco
1. Crea o abre la cuenta de comercio en https://dashboard.epayco.com
2. En **Configuración > Personalizaciones > Llaves secretas** copia:
   - `PUBLIC_KEY`
   - `PRIVATE_KEY`
   - `P_CUST_ID_CLIENTE`
   - `P_KEY`
3. En el panel, activa los medios de pago que quieras ofrecer: tarjeta, PSE, Nequi, Daviplata y efectivo.
   El cliente elige el método en la página y lo confirma en la ventana de ePayco.

## 2. Publicar en Vercel (gratis)
1. Crea una cuenta en https://vercel.com y sube esta carpeta completa. Puedes hacerlo de dos formas:
   - Importar un repositorio de GitHub que la contenga.
   - Ejecutar `npx vercel` dentro de la carpeta.
2. En **Project > Settings > Environment Variables** agrega:

   | Variable | Valor |
   |---|---|
   | `EPAYCO_PUBLIC_KEY` | tu PUBLIC_KEY |
   | `EPAYCO_PRIVATE_KEY` | tu PRIVATE_KEY |
   | `EPAYCO_P_CUST_ID_CLIENTE` | tu P_CUST_ID_CLIENTE (verifica las notificaciones) |
   | `EPAYCO_P_KEY` | tu P_KEY (verifica las notificaciones) |
   | `ORDER_SECRET` | una frase larga y secreta inventada por ti. Sella el monto de cada pedido; no la compartas. |
   | `EPAYCO_TEST` | `true` mientras pruebas |

3. Vuelve a desplegar (**Redeploy**).
4. Haz compras de prueba con las tarjetas de prueba de ePayco. Revisa estos cuatro casos: aprobado, rechazado, pendiente y cancelado.
5. Cuando todo funcione, cambia `EPAYCO_TEST` a `false` y vuelve a desplegar. Desde ese momento se cobra de verdad.

## 3. Cómo se protege cada pago
- **El total lo calcula el servidor** (`api/_epayco.js`), no el navegador.
- **Cada carrito tiene un número de pedido** y ePayco acepta un solo pago por número (`uniqueTransactionPerBill`). Así se evitan cobros duplicados.
- **Las notificaciones de ePayco llegan a `/api/epayco-confirmation`.** El servidor revisa la firma, la moneda y el sello del monto. Una notificación falsa o con otro monto se rechaza, y si la misma notificación llega varias veces, el resultado es el mismo.
- **La pantalla de "Pago realizado" solo aparece si `/api/epayco-status` verificó el pago con ePayco.** Llegar a la página de regreso no basta.

## 4. Transferencia y consignación
- Están **deshabilitadas** hasta que registres la cuenta del comercio en `BANK_ACCOUNT`, al inicio del script de `index.html`.
- Usa solo datos reales y autorizados.
- El pedido queda **pendiente de verificación** hasta que confirmes el ingreso del dinero en la cuenta. Un comprobante no lo da por pagado.

## 5. Cambiar precios o productos
Si cambias un precio o agregas un producto, cámbialo en **dos lugares**:
- `PRODUCTS` dentro de `index.html`, que es lo que ve el cliente.
- `CATALOG` dentro de `api/_epayco.js`, que es lo que se cobra.

## 6. Pedidos
Cada pago aparece en el panel de ePayco con:
- Los productos y sus cantidades, en `extra1`.
- La entrega y la dirección, en `extra2`.
- Los comentarios del cliente, en `extra3`.

Cada notificación verificada también queda en el registro (**Logs**) de Vercel.

**Pendiente:** para guardar los pedidos en un historial propio, o recibir un aviso automático, hace falta conectar una base de datos (por ejemplo Vercel KV / Upstash). Esto requiere tu autorización y credenciales.
