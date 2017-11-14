# Lightning Strike REST API

REST API for the integration of Lightning payments into e-commerce websites, built on top of c-lightning.

## Install & Setup

```bash
# install
$ git clone ... lightning-strike && cd lightning-strike
$ npm install

# configure
$ cp example.env .env && edit .env

# setup sqlite schema
$ knex migrate:latest

# run server
$ npm start
```

## API

Invoices have the following fields: `id`, `msatoshi`, `metadata`, `rhash`, `payreq`, `created_at`, `completed` and `completed_at`.

All endpoints accept and return data in JSON format.

### `POST /invoice`

Create a new invoice.

*Body parameters*: `msatoshi` and `metadata` (arbitrary order-related metadata, *optional*).

Returns `201 Created` and the invoice on success.

### `GET /invoices`

List invoices.

### `GET /invoice/:id`

Get the specified invoice.

### `GET /invoice/:id/wait?timeout=[sec]`

Long-polling invoice payment notification.

Waits for the invoice to be paid, then returns `200 OK` and the invoice.

If `timeout` (defaults to 30s) is reached before the invoice is paid, returns `402 Payment Required`.

### `POST /invoice/:id/webhook`

Register a URL as a web hook to be notified once the invoice is paid.

*Body parameters:* `url`.

Returns `201 Created` on success. Once the payment is made, an empty POST request will be made to the provided URL.

For security reasons, the provided `url` should contain a secret token used to verify the authenticity of the request (e.g. `HMAC(secret_key, rhash)`).

### `GET /payment-stream`

Returns live invoice payment updates as a [server-sent events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events) stream.

## Examples

```bash
# create new invoice
$ curl http://localhost:8009/invoice -X POST -d msatoshi=5000 -d metadata[customer_id]=9817 -d metadata[product_id]=7189

{"id":"07W98EUsBtCiyF7BnNcKe","msatoshi":"5000","metadata":{"customer_id":9817,"product_id":7189},"rhash":"3e449cc84d6b2b39df8e375d3cec0d2910e822346f782dc5eb97fea595c175b5","payreq":"lntb500n1pdq55z6pp58ezfejzddv4nnhuwxawnemqd9ygwsg35dauzm30tjll2t9wpwk6sdq0d3hz6um5wf5kkegcqpxpc06kpsp56fjh0jslhatp6kzmp8yxsgdjcfqqckdrrv0n840zqpx496qu5xenrzedlyatesl98dzdt5qcgkjd3l6vhax425jetq2h3gqz2enhk","completed":false,"created_at":1510625370087}

# ... with json
$ curl http://localhost:8009/invoice -X POST -H 'Content-Type: application/json' \
  -d '{"msatoshi":5000,"metadata":{"customer_id":9817,"products":[593,182]}

# fetch an invoice
$ curl http://localhost:8009/invoice/07W98EUsBtCiyF7BnNcKe

# fetch all invoices
$ curl http://localhost:8009/invoices

# register a web hook
$ curl http://localhost:8009/invoice/07W98EUsBtCiyF7BnNcKe/webhook -X POST -d url=https://requestb.in/pfqcmgpf

# long-poll payment notification for a specific invoice
$ curl http://localhost:8009/invoice/07W98EUsBtCiyF7BnNcKe/wait

# stream all incoming payments
$ curl http://localhost:8009/payment-stream
```

## License

MIT