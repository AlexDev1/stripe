import os

import requests
import stripe
from fastapi import HTTPException


@app.route('/create-checkout-session', methods=['POST'])
def create_checkout_session():
    try:
        checkout_session = stripe.checkout.Session.create(
            line_items=[
                {
                    # Provide the exact Price ID (for example, pr_1234) of the product you want to sell
                    'price': '{{PRICE_ID}}',
                    'quantity': 1,
                },
            ],
            mode='payment',
            success_url=YOUR_DOMAIN + '/success.html',
            cancel_url=YOUR_DOMAIN + '/cancel.html',
        )
    except Exception as e:
        return str(e)

    return redirect(checkout_session.url, code=303)

class AsyncStripePayGateway:
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.stripe_id = None
        self.status = None
        self.api_key = os.environ.get("STRIPE_SECRET_KEY")
        self.client_id = os.environ.get("STRIPE_PUBLISHABLE_KEY")
        self.domain = os.environ.get("STRIPE_DOMAIN")

    async def checkout_session(self, out_trade_no, body, total_amount, currency="USD", detail=None):
        domain_url = self.domain
        try:
            stripe.api_key = self.api_key
            checkout_session = stripe.checkout.Session.create(
                success_url=f'{domain_url}{"/profile?tab=score&session_id={CHECKOUT_SESSION_ID}"}',
                cancel_url=f'{domain_url}{"/create-poster"}',
                client_reference_id=out_trade_no,
                payment_method_types=["card"],
                mode="payment",
                # metadata=body,
                line_items=[{'price_data': {
                    'currency': currency,
                    'product_data': {
                        'name': body
                    },
                    'unit_amount': str(int(total_amount))
                }, "quantity": '1'
                }]
                # line_items=[{"price": total_amount, "quantity": 1}],
            )
            self.stripe_id = checkout_session.stripe_id
            self.url = checkout_session["url"]
        except Exception as e:
            raise HTTPException(status_code=403, detail=str(e))

    async def endpoint(self, request_data, stripe_signature):
        # You can use webhooks to receive information about asynchronous payment events.
        # For more about our webhook events check out https://stripe.com/docs/webhooks.
        webhook_secret = os.getenv('STRIPE_WEBHOOK_SECRET')
        stripe.api_key = self.api_key
        # request_data = json.loads(request.data)
        if webhook_secret:
            # Retrieve the event by verifying the signature using the raw body and secret if webhook signing is configured.
            signature = stripe_signature
            try:
                event = stripe.Webhook.construct_event(
                    payload=request_data, sig_header=signature, secret=webhook_secret)
            except Exception as e:
                event = None
        else:
            event = request_data.data
        return event
