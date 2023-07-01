## Intro

Google Analytics is a powerful web analytics tool that provides valuable insights into your website traffic and monitor user behavior. Integrating Google Analytics into your Next.js application allows you to track and analyze user interactions, measure marketing campaign effectiveness, and make data-driven decisions. We will explore the steps to set up Google Analytics (gtag.js) in a Next.js application.

## Google analytics account setup

- First, visit the [Google Analytics Website](https://analytics.google.com) and sign up using your google account.
- Start the Account setup then create a new property for your Next.js application and provide the details asked. After completing the setup, you will be taken to a new platform page for your account.

- Select the platform for your project, in this case that will be web.

![Image of choose platform page](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hv1m7dvjywuqmvo53m09.png)

- Enter the website url of where you next app is hosted and give a prefered name. Click on create stream to continue.

![website url and name page](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/cmrh1vrmatatj7llsaqo.png)

- Once the property stream created, you will be taken to a page similar to the one below. Note down the Property ID. It should be in the format G-XXXXXXXXX.

![Property ID page](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/huwyi1sgj05r70fvb380.png)

## Initialize Next.js project

Setup your Next.Js project as per [https://nextjs.org/](https://nextjs.org/) specifications.

This should work just fine:

```
npx create-next-app@latest your-next-app
```

## Environment variables

In the root directory of your project, you should have an environment `.env.local` file. If that is not the case, you can create a file by the same name. Add your property ID that we copied earlier here.

```
NEXT_PUBLIC_GOOGLE_ANALYTICS= 'G-XXXXXXXX'
```

## Create a google analytics Component

Create a components folder or any folder of your choice outside of the app directory and within your project directory, preferably within the `src` directory. In the components folder create a `GoogleAnalytics.tsx` file.
In the file copy the code below and save it.

```
import Script from "next/script";

const GoogleAnalytics = ({ ga_id }: { ga_id: string }) => (
  <>
    <Script
      async
      src={`https://www.googletagmanager.com/gtag/js? 
      id=${ga_id}`}
    ></Script>
    <Script
      id="google-analytics"
      dangerouslySetInnerHTML={{
        __html: `
          window.dataLayer = window.dataLayer || [];
          function gtag(){dataLayer.push(arguments);}
          gtag('js', new Date());
    
          gtag('config', '${ga_id}');
        `,
      }}
    ></Script>
  </>
);
export default GoogleAnalytics;

```

## Render the component in the Layout file

In the root `layout.tsx` file in the `app` directory, import your GoogleAnalytics component like so:

```
import GoogleAnalytics from "~/components/GoogleAnalytics";
```
You can then modify your code to match the format below.

```
export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
      <html lang="en">
        <body>
          {process.env.NEXT_PUBLIC_GOOGLE_ANALYTICS ? (
            <GoogleAnalytics ga_id= 
            {process.env.NEXT_PUBLIC_GOOGLE_ANALYTICS} />
          ) : null}
          // ... your other code content here.
        </body>
      </html>
  );
}
```
## Run your app

On your terminal, run the following to start your app.

```
npm run dev
```
On the browser, navigate to your next app live url, most likely it will be `http://localhost:3000`
Open the browser console.
On the console, type `datalayer`. If you see an output similar to the one below then, Yaay 🎉 you did it!

![image of output for typing datalayer](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hqpodk56nsr4ns4yyth4.png)

## Host your app

If you are planning to host your site, be sure to add your property in your hosting service project settings. You should find the environment variables settings and add you ID with the same keyname as used in the project `.env.local` file.

You can now go to the analytics dashboard and click on the live button, where you will see the information about the application in real-time!

For any follow up or questions, reach out through any of the following:
My [Personal website](collinsruto.netlify.app), [Twitter](https://twitter.com/Ruto_Collins_), [Github](https://github.com/Collins-Ruto) or [linkedin](https://www.linkedin.com/in/collins-ruto)

Sample github project:
[https://github.com/collins-ruto/learnhq](https://github.com/collins-ruto/learnhq)