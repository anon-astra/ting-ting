<script>
  import emailjs from '@emailjs/browser';
  function stored() { try { return JSON.parse(localStorage.getItem('ting-settings') || '{}') || {}; } catch { return {}; } }
  const initial = stored();
  let settings = $state({ service: typeof initial.service === 'string' ? initial.service : '', template: typeof initial.template === 'string' ? initial.template : '', key: typeof initial.key === 'string' ? initial.key : '' });
  let setup = $state(false), saved = $state(''), busy = $state(false), feedback = $state(''), failed = $state(false);
  let name = $state(''), email = $state(''), sender = $state(''), reply = $state('');
  let amount = $state(''), currency = $state('INR'), purpose = $state(''), due = $state(''), link = $state(''), note = $state(''), tone = $state('Friendly');
  let lastSent = $state('');
  let ready = $derived(Boolean(settings.service.trim() && settings.template.trim() && settings.key.trim()));
  let money = $derived(new Intl.NumberFormat('en-IN', { style: 'currency', currency }).format(Number(amount) || 0));
  let dateText = $derived(due ? new Date(due + 'T12:00:00').toLocaleDateString('en-IN', { day: 'numeric', month: 'short', year: 'numeric' }) : '');
  let subject = $derived(`Payment reminder${purpose.trim() ? ': ' + purpose.trim() : ''}`);
  let message = $derived(`Hi ${name.trim() || 'there'},\n\n${tone === 'Friendly' ? 'Just a friendly nudge about' : tone === 'Direct' ? 'This is a reminder for' : 'Please arrange payment for'} ${purpose.trim() || 'your payment'} of ${money}${dateText ? ', due on ' + dateText : ''}.${tone === 'Friendly' ? ' Whenever you get a moment, please take a look.' : tone === 'Firm' ? ' Please confirm once the payment is complete.' : ''}\n\n${note.trim() ? note.trim() + '\n\n' : ''}${link.trim() ? 'Payment link: ' + link.trim() + '\n\n' : ''}If you have already paid, please disregard this reminder.\n\nThanks,\n${sender.trim() || 'Your name'}`);
  function saveSettings(event) { event.preventDefault(); try { localStorage.setItem('ting-settings', JSON.stringify(settings)); saved = 'Saved on this device.'; } catch { saved = 'Settings work for this session. Browser storage is unavailable.'; } }
  async function send(event) {
    event.preventDefault(); if (busy) return;
    if (!ready) { setup = true; feedback = 'Connect EmailJS before sending.'; failed = true; return; }
    if (!(Number(amount) > 0) || !Number.isFinite(Number(amount))) { feedback = 'Enter a valid amount greater than zero.'; failed = true; return; }
    if (link.trim()) { try { if (new URL(link.trim()).protocol !== 'https:') throw new Error(); } catch { feedback = 'Use an HTTPS payment link.'; failed = true; return; } }
    busy = true; feedback = ''; const recipient = email.trim();
    try {
      await emailjs.send(settings.service.trim(), settings.template.trim(), { to_email: recipient, to_name: name.trim(), from_name: sender.trim(), reply_to: reply.trim(), subject, message, amount: money, due_date: dateText, payment_for: purpose.trim(), payment_link: link.trim() }, { publicKey: settings.key.trim(), limitRate: { id: 'ting-ting', throttle: 1500 } });
      lastSent = recipient; feedback = `Reminder accepted by EmailJS for ${recipient}.`; failed = false;
    } catch (error) { failed = true; feedback = error?.status === 429 ? 'Too many requests or quota reached. Please wait and check your EmailJS quota before retrying.' : 'EmailJS could not confirm sending. Check your connection, service, template and public key before retrying.'; }
    finally { busy = false; }
  }
</script>

<svelte:head><title>Ting Ting — A friendly nudge</title></svelte:head>
<div class="shell">
  <header><a class="brand" href="./" aria-label="Ting Ting home"><span class="brand-icon">↗</span> ting ting<span class="brand-dot">.</span></a><button class="connection" onclick={() => setup = !setup} aria-expanded={setup}><span class:connected={ready} class="dot"></span>{ready ? 'EmailJS configured' : 'Connect EmailJS'}<span>↗</span></button></header>
  {#if setup}
    <section class="settings" aria-label="EmailJS settings"><div class="section-heading"><div><span class="eyebrow">ONE-TIME SETUP</span><h2>Connect your email.</h2></div><button class="quiet" onclick={() => setup = false} aria-label="Close settings">✕</button></div><p>Connect an email service at <a href="https://dashboard.emailjs.com/" target="_blank" rel="noreferrer">EmailJS ↗</a>, then create a template with To email <code>{'{{to_email}}'}</code>, Subject <code>{'{{subject}}'}</code>, Reply to <code>{'{{reply_to}}'}</code> and body <code>{'{{message}}'}</code>. Use a preformatted text block to preserve line breaks.</p><form onsubmit={saveSettings}><div class="settings-fields"><label>Service ID<input required bind:value={settings.service} placeholder="service_…" /></label><label>Template ID<input required bind:value={settings.template} placeholder="template_…" /></label><label>Public key<input required bind:value={settings.key} placeholder="Your EmailJS public key" /></label></div><div class="settings-bottom"><small>Public key only. Settings stay in this browser.</small><button class="small-primary" type="submit">Save settings ↗</button></div><p class="saved" role="status">{saved}</p></form></section>
  {/if}
  <main>
    <section class="intro"><div class="eyebrow"><span class="tiny-line"></span> LESS AWKWARD. MORE PAID.</div><h1>A little nudge.<br/><span>A payment sorted.</span></h1><p>Skip the awkward follow-up. Send a thoughtful<br class="desktop"/> payment reminder in a few seconds.</p></section>
    <div class="workspace"><section class="composer"><div class="section-heading"><h2>New reminder</h2><span class="step">01 / COMPOSE</span></div>
      <form onsubmit={send}>
      <fieldset disabled={busy}>
      <div class="row"><label>Recipient name<input bind:value={name} required maxlength="100" placeholder="Who owes you?" autocomplete="off" /></label><label>Email address<input type="email" required bind:value={email} placeholder="hello@example.com" /></label></div>
      <div class="row amount-row"><label>Amount<div class="amount-input"><select aria-label="Currency" bind:value={currency}><option value="INR">₹ INR</option><option value="USD">$ USD</option><option value="EUR">€ EUR</option><option value="GBP">£ GBP</option></select><input aria-label="Amount" type="number" min="0.01" max="999999999" step="0.01" required bind:value={amount} placeholder="0.00" /></div></label><label>Due date <span class="optional">optional</span><input type="date" bind:value={due}/></label></div>
      <label>What's it for?<input bind:value={purpose} required maxlength="160" placeholder="Dinner split, project invoice, rent…" /></label>
      <div class="tone-label">Set the tone</div><div class="tones">{#each ['Friendly', 'Direct', 'Firm'] as choice}<button type="button" class:active={tone === choice} aria-pressed={tone === choice} onclick={() => tone = choice}>{choice === 'Friendly' ? '☺' : choice === 'Direct' ? '↗' : '！'} {choice}</button>{/each}</div>
      <div class="row"><label>Your name<input required bind:value={sender} maxlength="100" placeholder="Signed, you" autocomplete="name" /></label><label>Reply-to email<input type="email" required bind:value={reply} placeholder="you@example.com" autocomplete="email" /></label></div>
      <details><summary>Add a note or payment link <span>＋</span></summary><div class="extras"><label>Payment link <span class="optional">optional</span><input type="url" bind:value={link} placeholder="https://…" maxlength="2000" /></label><label>Personal note <span class="optional">optional</span><textarea rows="3" bind:value={note} maxlength="1500" placeholder="A little context goes a long way." /></label></div></details>
      <button class="send" type="submit">{busy ? 'Sending reminder…' : 'Send reminder'}<span>{busy ? '◌' : '↗'}</span></button><p class="send-caption">One reminder. Straight to their inbox.</p>
      </fieldset>
      </form>
      {#if feedback}<p class:error={failed} class="feedback" role={failed ? 'alert' : 'status'}>{feedback}</p>{/if}
    </section>
    <aside><div class="preview-label"><span class="step">02 / THE PREVIEW</span><span class="live"><span class="dot connected"></span> Live preview</span></div><section class="preview"><div class="email-meta"><div><span>TO</span><strong>{email || 'recipient@example.com'}</strong></div><div><span>SUBJECT</span><strong>{subject}</strong></div></div><div class="email-body"><div class="mini-brand">ting ting<span>.</span></div><span class="eyebrow">PAYMENT REMINDER</span><div class="preview-amount">{money}</div><div class="preview-for">{purpose || 'Your payment'}{#if dateText}<span>Due {dateText}</span>{/if}</div><pre>{message}</pre></div><div class="preview-footer"><span class="dot connected"></span> A small ping. A little peace of mind.</div></section><p class="preview-note">Message preview · Email appearance follows your EmailJS template.</p><div class="tip"><span>↗</span><p><strong>Good reminders keep it simple.</strong><br/>A clear amount, a little context, and a friendly tone.</p></div>{#if lastSent}<div class="last-sent">✓ Last reminder submitted to <strong>{lastSent}</strong></div>{/if}</aside></div>
  </main><footer><span>Small nudges. Better days.</span><span>Made to keep things simple <span class="brand-dot">↗</span></span></footer>
</div>
