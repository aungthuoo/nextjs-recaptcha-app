# Using Google's ReCaptcha V3 with NextJS 13 and the New App Router 

## Quick notes
### React.js installation 
```sh
npx create-next-app@latest my-recaptcha-app
cd my-recaptcha-app

npm install axios react-google-recaptcha-v3
npm run dev 
```

### Generate Google ReCaptcha V3 
```sh 
https://www.google.com/recaptcha/about/
```

#### .env.local or .env 
```sh 
NEXT_PUBLIC_RECAPTCHA_SITE_KEY=6LeFZTIoAAAAA...
RECAPTCHA_SECRET_KEY=6LeFZTIoAAAA...
```

### Folder structure 
```sh
- app/
  - api/
    - contactFormSubmit/
      - route.ts
  - public/
  - page.tsx
  - google-captcha-wrapper.tsx
```
### Full Code Listings

### `/app/page.tsx`

```js 
"use client";

import React, { useState } from "react";
import { useGoogleReCaptcha } from "react-google-recaptcha-v3";
import axios from "axios";
import GoogleCaptchaWrapper from "@/app/google-captcha-wrapper";

interface PostData {
  gRecaptchaToken: string;
  firstName: string;
  lastName: string;
  email: string;
  hearFromSponsors: boolean;
}

export default function Home() {
  return (
    <GoogleCaptchaWrapper>
      <HomeInside />
    </GoogleCaptchaWrapper>
  );
}

function HomeInside() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');
  const [email, setEmail] = useState('');
  const [hearFromSponsors, setHearFromSponsors] = useState(false);
  const [notification, setNotification] = useState('');

  const { executeRecaptcha } = useGoogleReCaptcha();

  const handleSubmitForm = function (e: any) {
    e.preventDefault();
    if (!executeRecaptcha) {
      console.log("Execute recaptcha not available yet");
      setNotification(
        "Execute recaptcha not available yet likely meaning key not recaptcha key not set"
      );
      return;
    }
    executeRecaptcha("enquiryFormSubmit").then((gReCaptchaToken) => {
      submitEnquiryForm(gReCaptchaToken);
    });
  };

  const submitEnquiryForm = (gReCaptchaToken : string) => {
    async function goAsync() {
      const response = await axios({
        method: "post",
        url: "/api/contactFormSubmit",
        data: {
          firstName: firstName,
          lastName: lastName,
          email: email,
          hearFromSponsors: hearFromSponsors,
          gRecaptchaToken: gReCaptchaToken,
        },
        headers: {
          Accept: "application/json, text/plain, */*",
          "Content-Type": "application/json",
        },
      });


      if (response?.data?.success === true) {
        setNotification(`Success with score: ${response?.data?.score}`);
      } else {
        setNotification(`Failure with score: ${response?.data?.score}`);
      }
    }
    goAsync().then(() => {}); // suppress typescript error
  };

  return (
    <div className="container">
      <main className="mt-5"> {/* Add a top margin for better spacing */}
        <h2>Interested in Silicon Valley Code Camp</h2>
        <form onSubmit={handleSubmitForm}>
          <div className="mb-3">
            <input
              type="text"
              name="firstName"
              value={firstName}
              onChange={(e) => setFirstName(e?.target?.value)}
              className="form-control"
              placeholder="First Name"
            />
          </div>
          <div className="mb-3">
            <input
              type="text"
              name="lastName"
              value={lastName}
              onChange={(e) => setLastName(e?.target?.value)}
              className="form-control"
              placeholder="Last Name"
            />
          </div>
          <div className="mb-3">
            <input
              type="text"
              name="email"
              value={email}
              onChange={(e) => setEmail(e?.target?.value)}
              className="form-control"
              placeholder="Email Address"
            />
          </div>
          <div className="mb-3 form-check">
            <input
              type="checkbox"
              name="hearFromSponsors"
              checked={hearFromSponsors}
              onChange={(e) => setHearFromSponsors(e?.target?.checked)}
              className="form-check-input"
            />
            <label className="form-check-label">Hear from our sponsors</label>
          </div>
          <button type="submit" className="btn btn-light">Submit</button>
          {notification && <p className="mt-3 text-info">{notification}</p>}
        </form>
      </main>
    </div>
  );
}
```


### `/app/google-captcha-wrapper.tsx`

```js
"use client";
import { GoogleReCaptchaProvider } from "react-google-recaptcha-v3";
import React from "react";

export default function GoogleCaptchaWrapper({
  children,
}: {
  children: React.ReactNode;
}) {
  const recaptchaKey: string | undefined =
    process?.env?.NEXT_PUBLIC_RECAPTCHA_SITE_KEY;
  return (
    <GoogleReCaptchaProvider
      reCaptchaKey={recaptchaKey ?? "NOT DEFINED"}
      scriptProps={{
        async: false,
        defer: false,
        appendTo: "head",
        nonce: undefined,
      }}
    >
      {children}
    </GoogleReCaptchaProvider>
  );
}

```


### `/app/api/contactFormSubmit/route.ts`

```js
import { NextResponse } from "next/server";
import axios from "axios";

export async function POST(request: Request, response: Response) {
  const secretKey = process?.env?.RECAPTCHA_SECRET_KEY;

  const postData = await request.json();
  const { gRecaptchaToken, firstName, lastName, email, hearFromSponsors } =
    postData;

  console.log(
    "gRecaptchaToken,firstName,lastName,email,hearFromSponsors:",
    gRecaptchaToken?.slice(0, 10) + "...",
    firstName,
    lastName,
    email,
    hearFromSponsors
  );

  let res: any;
  const formData = `secret=${secretKey}&response=${gRecaptchaToken}`;
  try {
    res = await axios.post(
      "https://www.google.com/recaptcha/api/siteverify",
      formData,
      {
        headers: {
          "Content-Type": "application/x-www-form-urlencoded",
        },
      }
    );
  } catch (e) {
    console.log("recaptcha error:", e);
  }

  if (res && res.data?.success && res.data?.score > 0.5) {
    // Save data to the database from here
    console.log("Saving data to the database:", firstName, lastName, email, hearFromSponsors);
    console.log("res.data?.score:", res.data?.score);

    return NextResponse.json({
      success: true,
      firstName, lastName,
      score: res.data?.score,
    });
  } else {
    console.log("fail: res.data?.score:", res.data?.score);
    return NextResponse.json({ success: false, name, score: res.data?.score });
  }
}
```









# More detail explaination

Introduction
------------

CAPTCHA, which stands for “Completely Automated Public Turing test to tell Computers and Humans Apart,” is a way to differentiate between human users and bots. This functionality is especially crucial for form submissions to prevent spam and abuse. Invisible CAPTCHA doesn’t interrupt the user flow and performs these checks seamlessly in the background.

Google ReCaptcha V3
-------------------

Google ReCaptcha V3 is an excellent example of an invisible CAPTCHA. After you include some JavaScript from Google on your page, that script will add a token to your form submission. Then, on the server-side, you’ll verify this token using a secret key that only the server knows. If the CAPTCHA verification passes, you’re clear to proceed with operations like registering a user.

### How It Works

1.  Include Google’s ReCaptcha JavaScript on your webpage.
2.  On form submit, a token is added automatically by the ReCaptcha script.
3.  Send this token along with your form data to the server.
4.  Server-side: verify this token using Google’s secret key.
5.  If it passes, proceed with further processing.

NextJS 13’s App Router
----------------------

NextJS 13 introduces a new folder structure, notably different from the older `pages` directory approach. The new structure looks something like this:

```sh
- app/
  - api/
    - contactFormSubmit/
      - route.ts
  - public/
  - page.tsx
  - google-captcha-wrapper.tsx
```


Here, `/app/api/contactFormSubmit/route.ts` is where your server-side logic resides. Specifically, this is the server-side handler that listens to HTTP POST requests from the form on your website. This form submission will include a token generated by Google ReCaptcha V3.

This co-location feature allows your server-side logic to reside closer to your client-side logic. Essentially, you can have server-side and client-side logic co-located, making your project easier to navigate and manage.

Setting Up Google ReCaptcha V3 and Environment Variables
--------------------------------------------------------

Before we deep-dive into the code, it’s crucial to register your app on Google’s Developer Portal and get the keys for ReCaptcha V3.

1.  **Google Developer Portal**: Head over to the [Google ReCaptcha Website](https://www.google.com/recaptcha) and click on the ‘Admin Console’ button. Sign in with your Google account if you haven’t already.
    
2.  **Create a New Site**: Once in the console, hit the ’+’ button to create a new site. Choose “ReCaptcha V3”, give your domain, and note down the keys you’ll receive. There are two keys: one is the site key and the other is the secret key.
    

Now, let’s bring those keys into our Next.js application environment.

**Creating .env file**

In the root directory of your NextJS project, create a new file named `.env`. This is where we’ll place our keys securely. Your `.env` file will look something like this:

```
NEXT_PUBLIC_RECAPTCHA_SITE_KEY=6LeFZTIoAAAAA...
RECAPTCHA_SECRET_KEY=6LeFZTIoAAAA...
```


In our codebase, you’ll notice that these keys are read through `process.env.NEXT_PUBLIC_RECAPTCHA_SITE_KEY` and `process.env.RECAPTCHA_SECRET_KEY`.

**Note**: Prefixing an environment variable with `NEXT_PUBLIC_` exposes it to the client-side code. So, `NEXT_PUBLIC_RECAPTCHA_SITE_KEY` is accessible in your client-side components. It’s a naming convention Next.js uses to inject runtime environment variables.

That’s it! You’ve set up the Google Developer Portal and safely stored the environment variables for use in your Next.js application. Now, let’s plug these keys into our code.

Implementation Example
----------------------

To use the code snippets below, you’ll need to install the npm package `react-google-recaptcha-v3`.

Also, if you want to try this example yourself, all the code can be found at this URL: [https://github.com/pkellner/nextjs-google-recaptcha-v3-app-router-demo](https://github.com/pkellner/nextjs-google-recaptcha-v3-app-router-demo)

### Client-side Code

In `/app/page.tsx`, use the `use client` directive to run this file in the browser. Omitting this directive would treat the component as a server component, which is beyond the scope of this article.

Here’s the code that runs in your browser:

```sh 
// Code for /app/page.tsx (see below for full code listing)
```


And it’s wrapped with this provider:

```sh
// Code for GoogleCaptchaWrapper (see below for full code listing)
```


Note that `GoogleReCaptchaProvider` must wrap any component that uses the hook `useGoogleReCaptcha`. The `HomeInside` component is required for better code organization.

### Server-side Code

Here is the handler code that runs when the user submits the form:

```
// Code for /app/api/contactFormSubmit/route.ts (see below for full code listing)
```


The server-side handler listens for a `POST` request at `http://localhost:3000/api/contactFormSubmit` when you’re running the app locally.

Wrapping Up
-----------

You’ve made it to the end, and what a journey it’s been! We’ve delved into the intricacies of Google ReCaptcha V3, explored Invisible CAPTCHA, and even got our hands dirty with NextJS 13’s new App Router. The code snippets should guide you in crafting a smoother and more secure user experience, which is always a win-win.

If you’ve enjoyed this read as much as I’ve enjoyed writing it, that’s a success in my book. Until next time, code on! 🚀

Full Code Listings
------------------

### `/app/page.tsx`

```js 
"use client";

import React, { useState } from "react";
import { useGoogleReCaptcha } from "react-google-recaptcha-v3";
import axios from "axios";
import GoogleCaptchaWrapper from "@/app/google-captcha-wrapper";

interface PostData {
  gRecaptchaToken: string;
  firstName: string;
  lastName: string;
  email: string;
  hearFromSponsors: boolean;
}

export default function Home() {
  return (
    <GoogleCaptchaWrapper>
      <HomeInside />
    </GoogleCaptchaWrapper>
  );
}

function HomeInside() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');
  const [email, setEmail] = useState('');
  const [hearFromSponsors, setHearFromSponsors] = useState(false);
  const [notification, setNotification] = useState('');

  const { executeRecaptcha } = useGoogleReCaptcha();

  const handleSubmitForm = function (e: any) {
    e.preventDefault();
    if (!executeRecaptcha) {
      console.log("Execute recaptcha not available yet");
      setNotification(
        "Execute recaptcha not available yet likely meaning key not recaptcha key not set"
      );
      return;
    }
    executeRecaptcha("enquiryFormSubmit").then((gReCaptchaToken) => {
      submitEnquiryForm(gReCaptchaToken);
    });
  };

  const submitEnquiryForm = (gReCaptchaToken : string) => {
    async function goAsync() {
      const response = await axios({
        method: "post",
        url: "/api/contactFormSubmit",
        data: {
          firstName: firstName,
          lastName: lastName,
          email: email,
          hearFromSponsors: hearFromSponsors,
          gRecaptchaToken: gReCaptchaToken,
        },
        headers: {
          Accept: "application/json, text/plain, */*",
          "Content-Type": "application/json",
        },
      });


      if (response?.data?.success === true) {
        setNotification(`Success with score: ${response?.data?.score}`);
      } else {
        setNotification(`Failure with score: ${response?.data?.score}`);
      }
    }
    goAsync().then(() => {}); // suppress typescript error
  };

  return (
    <div className="container">
      <main className="mt-5"> {/* Add a top margin for better spacing */}
        <h2>Interested in Silicon Valley Code Camp</h2>
        <form onSubmit={handleSubmitForm}>
          <div className="mb-3">
            <input
              type="text"
              name="firstName"
              value={firstName}
              onChange={(e) => setFirstName(e?.target?.value)}
              className="form-control"
              placeholder="First Name"
            />
          </div>
          <div className="mb-3">
            <input
              type="text"
              name="lastName"
              value={lastName}
              onChange={(e) => setLastName(e?.target?.value)}
              className="form-control"
              placeholder="Last Name"
            />
          </div>
          <div className="mb-3">
            <input
              type="text"
              name="email"
              value={email}
              onChange={(e) => setEmail(e?.target?.value)}
              className="form-control"
              placeholder="Email Address"
            />
          </div>
          <div className="mb-3 form-check">
            <input
              type="checkbox"
              name="hearFromSponsors"
              checked={hearFromSponsors}
              onChange={(e) => setHearFromSponsors(e?.target?.checked)}
              className="form-check-input"
            />
            <label className="form-check-label">Hear from our sponsors</label>
          </div>
          <button type="submit" className="btn btn-light">Submit</button>
          {notification && <p className="mt-3 text-info">{notification}</p>}
        </form>
      </main>
    </div>
  );
}
```


### `/app/google-captcha-wrapper.tsx`

```js
"use client";
import { GoogleReCaptchaProvider } from "react-google-recaptcha-v3";
import React from "react";

export default function GoogleCaptchaWrapper({
  children,
}: {
  children: React.ReactNode;
}) {
  const recaptchaKey: string | undefined =
    process?.env?.NEXT_PUBLIC_RECAPTCHA_SITE_KEY;
  return (
    <GoogleReCaptchaProvider
      reCaptchaKey={recaptchaKey ?? "NOT DEFINED"}
      scriptProps={{
        async: false,
        defer: false,
        appendTo: "head",
        nonce: undefined,
      }}
    >
      {children}
    </GoogleReCaptchaProvider>
  );
}

```


### `/app/api/contactFormSubmit/route.ts`

```js
import { NextResponse } from "next/server";
import axios from "axios";

export async function POST(request: Request, response: Response) {
  const secretKey = process?.env?.RECAPTCHA_SECRET_KEY;

  const postData = await request.json();
  const { gRecaptchaToken, firstName, lastName, email, hearFromSponsors } =
    postData;

  console.log(
    "gRecaptchaToken,firstName,lastName,email,hearFromSponsors:",
    gRecaptchaToken?.slice(0, 10) + "...",
    firstName,
    lastName,
    email,
    hearFromSponsors
  );

  let res: any;
  const formData = `secret=${secretKey}&response=${gRecaptchaToken}`;
  try {
    res = await axios.post(
      "https://www.google.com/recaptcha/api/siteverify",
      formData,
      {
        headers: {
          "Content-Type": "application/x-www-form-urlencoded",
        },
      }
    );
  } catch (e) {
    console.log("recaptcha error:", e);
  }

  if (res && res.data?.success && res.data?.score > 0.5) {
    // Save data to the database from here
    console.log("Saving data to the database:", firstName, lastName, email, hearFromSponsors);
    console.log("res.data?.score:", res.data?.score);

    return NextResponse.json({
      success: true,
      firstName, lastName,
      score: res.data?.score,
    });
  } else {
    console.log("fail: res.data?.score:", res.data?.score);
    return NextResponse.json({ success: false, name, score: res.data?.score });
  }
}
```
