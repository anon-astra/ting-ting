# Ting Ting

A pitch-black, green-accented Svelte 5 app for sending payment reminders through EmailJS. Responsive layout, live message preview, INR by default, optional due date and HTTPS payment link, and three reminder tones.

## Run

Node.js 22.12+ is recommended.

```sh
npm install
npm run dev
```

`npm run check` checks Svelte components. `npm run build` creates the static site in `dist/`. The GitHub Actions build checks the app and uploads the built site as an artifact. Vite uses a relative base, so the output supports a subdirectory such as `/ting-ting/`.

## EmailJS setup

1. Create an account at https://dashboard.emailjs.com/ and connect an email service.
2. Create an email template. Configure these template fields exactly:

| EmailJS field | Value |
| --- | --- |
| To Email | `{{to_email}}` |
| From Name | `{{from_name}}` |
| From Email | Your connected email service's verified/default sender |
| Reply To | `{{reply_to}}` |
| Subject | `{{subject}}` |

Use this HTML body in the template's code editor. Double braces escape user input; do not use triple braces.

```html
<div style="font-family:Arial,sans-serif;font-size:15px;line-height:1.7;color:#222">
  <pre style="white-space:pre-wrap;font-family:Arial,sans-serif;margin:0">{{message}}</pre>
</div>
```

3. Open **Connect EmailJS** in Ting Ting. Enter your service ID, template ID and **public** key, then save. Do not enter a private key or email password.
4. Add your deployed site's origin to EmailJS's allowed origins. Settings are stored only in this browser; each device must configure its own account. The form's recipient and message are not saved to local storage.
5. Fill in the reminder, check the live preview and click **Send reminder**.

The app sends `to_email`, `to_name`, `from_name`, `reply_to`, `subject`, `message`, `amount`, `due_date`, `payment_for` and `payment_link` to your template. The preview shows the message content; the EmailJS template determines the actual email appearance.

Sending is manual. A due date is message content, not a scheduled send. An accepted request does not guarantee inbox delivery. Loading state prevents duplicate clicks, and failures remain visible without silently retrying. No real emails are sent by CI.

## Hosting

Deploy `dist/` to any static host. For GitHub Pages, set Pages source to GitHub Actions, then add a Pages deployment workflow or deploy the built artifact. No backend or private API credentials are required.
