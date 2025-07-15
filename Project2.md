### 1.Add axiosSecure :
```js
import React from 'react';
import axios from 'axios'

const useAxiosSesure = () => {

  const instance = axios.create({
  baseURL: `http://localhost:3000`
});
  return instance
    
  }
export default useAxiosSesure;



//use
  axiosSecure.post("/parcels",ParcelData)
    .then(res=>{
      console.log(res.data);
    })

```
### 2.Tanstack Quary :
```js
1.npm i @tanstack/react-query
2.main.jsx add :
import {
  QueryClient,
  QueryClientProvider,
} from '@tanstack/react-query';

const queryClient = new QueryClient()
<QueryClientProvider client={queryClient}>
      //AuthProveider
    </QueryClientProvider>
```
### 3. TanStack Use :
```js
 const {UserData} = useAuth()
  const axiosSecure = useAxiosSesure()
  const {data:parcels}=useQuery({
    queryKey :["my-percel", UserData?.email],
    queryFn :async()=>{
      const res=await axiosSecure.get(`/parcels?email=${UserData?.email}`)
      return res.data
    }
  })
  console.log(parcels);
```
### Stripe Payment GetWay :
```js
1.npm install @stripe/react-stripe-js @stripe/stripe-js

```
### Payment file
```js
2.import { Elements } from "@stripe/react-stripe-js";
import { loadStripe } from '@stripe/stripe-js';
import PaymentFrom from "./PaymentFrom";


const stripePromise = loadStripe("pk_test_51RhpkKRh9Db7MzPj1GmZV5NgN5b7Hlsbvu4APLQIpvz4muwImMc09T02YSnQgggseHnLCDlvvgxyj3zv2GJvViuE00bHRp5fIo"); // Replace with your actual publishable key

const Payment = () => {
  return (
    <div className="p-4 max-w-xl mx-auto">
      <h2 className="text-2xl font-bold mb-4">Make a Payment</h2>
      <Elements stripe={stripePromise}>
        <PaymentFrom></PaymentFrom>
      </Elements>
    </div>
  );
};

export default Payment;


```
### paymentForm
```js
import { CardElement, useElements, useStripe } from '@stripe/react-stripe-js';
import { useQuery } from '@tanstack/react-query';
import { useState } from 'react';
import { useNavigate, useParams } from 'react-router-dom';
import Swal from 'sweetalert2';
import useAuth from '../../Hook/UseAuth';
import useAxiosSesure from '../../Hook/useAxiosSesure';

const PaymentForm = () => {
  const { id } = useParams();
  const navigate = useNavigate();
  const stripe = useStripe();
  const elements = useElements();
  const axiosSecure = useAxiosSesure();
  const { UserData } = useAuth();

  const [error, setError] = useState('');
  const [processing, setProcessing] = useState(false);
  const [successMessage, setSuccessMessage] = useState('');

  // 🔄 Fetch parcel data by ID
  const { data: parcel, isLoading } = useQuery({
    queryKey: ['parcel', id],
    queryFn: async () => {
      const res = await axiosSecure.get(`/parcels/${id}`);
      return res.data;
    },
    enabled: !!id,
  });

  // 🔒 Handle Payment Submission
  const handleSubmit = async (e) => {
    e.preventDefault();
    setProcessing(true);
    setError('');
    setSuccessMessage('');

    if (!stripe || !elements) {
      setError('Stripe is not loaded');
      setProcessing(false);
      return;
    }

    const card = elements.getElement(CardElement);
    if (!card) {
      setError('Card element not found');
      setProcessing(false);
      return;
    }

    try {
      // Step 1: Create payment method
      const { error: methodError, paymentMethod } = await stripe.createPaymentMethod({
        type: 'card',
        card,
      });

      if (methodError) {
        setError(methodError.message);
        setProcessing(false);
        return;
      }

      // Step 2: Create payment intent on server
      const { data } = await axiosSecure.post('/create-payment-intent', {
        amountInCents: parcel.Cost * 100,
        parcelId: id,
      });

      const clientSecret = data.clientSecret;

      // Step 3: Confirm card payment with Stripe
      const { error: confirmError, paymentIntent } = await stripe.confirmCardPayment(clientSecret, {
        payment_method: {
          card,
          billing_details: {
            name: UserData?.displayName || 'Unknown',
            email: UserData?.email || 'no-email@example.com',
          },
        },
      });

      if (confirmError) {
        setError(confirmError.message);
        return;
      }

      if (paymentIntent.status === 'succeeded') {
        setSuccessMessage('✅ Payment successful!');

        const paymentData = {
          parcelId: id,
          email: UserData.email,
          amount: parcel.Cost,
          transactionId: paymentIntent.id,
          paymentMethod: paymentIntent.payment_method_types,
        };

        // Step 4: Store payment in DB
        const paymentRes = await axiosSecure.post('/payments', paymentData);
        if (paymentRes.data.insertedId) {
          await Swal.fire({
            icon: 'success',
            title: 'Payment Successful!',
            html: `<strong>Transaction ID:</strong> <code>${paymentIntent.id}</code>`,
            confirmButtonText: 'Go to My Parcels',
          });

          navigate('/dashboard/mypercel');
        }
      }
    } catch (err) {
      console.error('❌ Payment error:', err);
      setError('Something went wrong. Please try again.');
    } finally {
      setProcessing(false);
    }
  };

  return (
    <div className="max-w-md mx-auto bg-white shadow-md rounded p-6">
      <h2 className="text-xl font-semibold mb-4">Pay for Your Parcel</h2>

      {isLoading ? (
        <p className="text-gray-600">Loading parcel details...</p>
      ) : (
        <div className="mb-4 space-y-1">
          <p><span className="font-medium">Created By:</span> {parcel?.Created_by}</p>
          <p><span className="font-medium">Cost:</span> ${parcel?.Cost}</p>
        </div>
      )}

      <form onSubmit={handleSubmit} className="space-y-4">
        <CardElement
          options={{
            style: {
              base: {
                fontSize: '16px',
                color: '#424770',
                '::placeholder': { color: '#aab7c4' },
              },
              invalid: { color: '#9e2146' },
            },
          }}
          className="p-3 border border-gray-300 rounded"
        />

        <button
          type="submit"
          disabled={!stripe || processing || isLoading}
          className="w-full bg-blue-600 text-white py-2 rounded hover:bg-blue-700 transition"
        >
          {processing ? 'Processing...' : `Pay $${parcel?.Cost || ''}`}
        </button>

        {error && <p className="text-red-500 text-sm">{error}</p>}
        {successMessage && <p className="text-green-600 text-sm">{successMessage}</p>}
      </form>
    </div>
  );
};

export default PaymentForm;

```
### Payment History
```js
import React from 'react';

import { useQuery } from '@tanstack/react-query';
import useAuth from '../../Hook/UseAuth';
import useAxiosSesure from '../../Hook/useAxiosSesure';


const formatDate = (iso) => new Date(iso).toLocaleString();

const PaymentHistory = () => {
    const { UserData } = useAuth();
    const axiosSecure = useAxiosSesure();

    const { isPending, data: payments = [] } = useQuery({
        queryKey: ['payments', UserData.email],
        queryFn: async () => {
            const res = await axiosSecure.get(`/payments?email=${UserData.email}`);
            return res.data;
        }
    })

    if (isPending) {
        return '...loading'
    }

    return (
        <div className="overflow-x-auto shadow-md rounded-xl">
            <table className="table table-zebra w-full">
                <thead className="bg-base-200 text-base font-semibold">
                    <tr>
                        <th>#</th>
                        <th>Parcel ID</th>
                        <th>Amount</th>
                        <th>Transaction</th>
                        <th>Paid At</th>
                    </tr>
                </thead>
                <tbody>
                    {payments?.length > 0 ? (
                        payments.map((p, index) => (
                            <tr key={p.transactionId}>
                                <td>{index + 1}</td>
                                <td className="truncate" title={p.parcelId}>
                                    {p.parcelId}...
                                </td>
                                <td>৳{p.amount}</td>
                                <td className="font-mono text-sm">
                                    <span title={p.transactionId}>
                                        {p.transactionId}...
                                    </span>
                                </td>
                                <td>{formatDate(p.paid_at_string)}</td>
                            </tr>
                        ))
                    ) : (
                        <tr>
                            <td colSpan="7" className="text-center text-gray-500 py-6">
                                No payment history found.
                            </td>
                        </tr>
                    )}
                </tbody>
            </table>
        </div>
    );
};

export default PaymentHistory;
```
## server
```js
1. npm i Stripe
2. const Stripe = require('stripe');
const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);

3.
 app.post('/create-payment-intent', async (req, res) => {
      try {
        const { amountInCents } = req.body;

        if (!amountInCents || amountInCents < 1) {
          return res.status(400).json({ error: 'Invalid amount' });
        }

        const paymentIntent = await stripe.paymentIntents.create({
          amount: amountInCents,
          currency: 'usd',
          payment_method_types: ['card'],
        });

        res.send({ clientSecret: paymentIntent.client_secret });
      } catch (error) {
        console.error("Stripe Payment Error:", error);
        res.status(500).json({ error: error.message });
      }
    });



```
### payment stauts update:
```js
app.post('/payments', async (req, res) => {
      try {
        const { parcelId, email, amount, paymentMethod, transactionId } = req.body;

        // 1. Update parcel's payment_status
        const updateResult = await ParcelCollection.updateOne(
          { _id: new ObjectId(parcelId) },
          {
            $set: {

              Status: 'paid'
            }
          }
        );

        if (updateResult.modifiedCount === 0) {
          return res.status(404).send({ message: 'Parcel not found or already paid' });
        }

        // 2. Insert payment record
        const paymentDoc = {
          parcelId,
          email,
          amount,
          paymentMethod,
          transactionId,
          paid_at_string: new Date().toISOString(),
          paid_at: new Date(),
        };

        const paymentResult = await paymentsCollection.insertOne(paymentDoc);

        res.status(201).send({
          message: 'Payment recorded and parcel marked as paid',
          insertedId: paymentResult.insertedId,
        });

      } catch (error) {
        console.error('Payment processing failed:', error);
        res.status(500).send({ message: 'Failed to record payment' });
      }
    });

```
### Payment History
```js
 app.get('/payments', async (req, res) => {
            try {
                const userEmail = req.query.email;

                const query = userEmail ? { email: userEmail } : {};
                const options = { sort: { paid_at: -1 } }; // Latest first

                const payments = await paymentsCollection.find(query, options).toArray();
                res.send(payments);
            } catch (error) {
                console.error('Error fetching payment history:', error);
                res.status(500).send({ message: 'Failed to get payments' });
            }
        });

```
