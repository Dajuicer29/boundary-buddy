# 🔧 Technical Deployment Guide

## Step 1: Push to GitHub

### 1.1 Create GitHub Account (if needed)
```
1. github.com → Sign up
2. Use email: your-email@gmail.com
3. Username: your-username
4. Create account
```

### 1.2 Create Repository

**Via GitHub Web:**
```
1. github.com → Click "+" → New repository
2. Name: boundary-buddy
3. Description: "Track and strengthen your boundaries"
4. Make public (better for visibility)
5. Click "Create repository"
```

**Via Command Line:**
```bash
# In your project directory
git remote add origin https://github.com/YOUR-USERNAME/boundary-buddy.git
git branch -M main
git push -u origin main
```

### 1.3 Verify Code is on GitHub
```
Visit: https://github.com/YOUR-USERNAME/boundary-buddy
Should see all files (index.html, README.md, etc)
```

---

## Step 2: Deploy to Vercel (Recommended)

### 2.1 Create Vercel Account
```
1. vercel.com → Sign up with GitHub
2. Authorize Vercel to access your GitHub
3. Done!
```

### 2.2 Import Project

**In Vercel Dashboard:**
```
1. Dashboard → New Project
2. Select "boundary-buddy" repository
3. Framework: Leave blank (static site)
4. Root directory: . (current)
5. Click "Deploy"
```

**Deployment starts automatically:**
```
- Build: ~10 seconds
- Deploy: ~5 seconds
- Live site: https://boundary-buddy.vercel.app
```

### 2.3 Connect Custom Domain

**Once domain is purchased:**

```
Vercel Dashboard:
1. Project → Settings → Domains
2. Click "Add Domain"
3. Enter: boundary-buddy.app (or your domain)
4. Vercel shows nameserver info
```

**Update domain registrar (Namecheap example):**
```
1. Namecheap Dashboard → Your Domains
2. Manage → Nameservers
3. Custom DNS
4. Add Vercel nameservers:
   - ns1.vercel-dns.com
   - ns2.vercel-dns.com
5. Save
6. Wait 5-30 minutes for DNS propagation
```

**Verify domain connected:**
```
Vercel Dashboard → Domains
Should show: boundary-buddy.app ✓ (verified)
```

---

## Step 3: Deploy to Netlify (Alternative)

### 3.1 Create Netlify Account
```
1. netlify.com → Sign up with GitHub
2. Authorize Netlify
3. Done!
```

### 3.2 Create Site

**In Netlify:**
```
1. Dashboard → New site from Git
2. Select GitHub
3. Choose "boundary-buddy" repository
4. Build settings:
   - Build command: (leave empty)
   - Publish directory: .
5. Click "Deploy site"
```

**Site goes live at:**
```
https://friendly-name.netlify.app
(Netlify generates a random name)
```

### 3.3 Connect Custom Domain

**In Netlify:**
```
1. Domain settings → Custom domains
2. Add custom domain: boundary-buddy.app
3. Update nameservers (same as Vercel)
```

---

## Step 4: Setup Stripe (Payments)

### 4.1 Create Stripe Account
```
1. stripe.com → Get Started
2. Enter email
3. Verify email
4. Complete account setup
5. Verify business (adds trust)
```

### 4.2 Get API Keys

**In Stripe Dashboard:**
```
1. Developers → API Keys
2. Copy Publishable key (starts with pk_live_)
3. Copy Secret key (starts with sk_live_)
4. SAVE THESE SAFELY (use environment variables)
```

### 4.3 Create Payment Product

**Option A: Payment Link (Easiest)**
```
Stripe Dashboard:
1. Products → New product
2. Name: "Boundary Buddy Premium"
3. Price: 4.99 USD
4. Billing period: Monthly
5. Save
6. Create payment link
7. Copy link
```

**Update upgradePremium() in index.html:**
```javascript
function upgradePremium() {
  window.location.href = 'https://buy.stripe.com/YOUR_PAYMENT_LINK_HERE';
}
```

**Option B: Checkout Session (More Control)**

Create a serverless function (for Vercel/Netlify):

**File: /api/stripe-checkout.js** (Vercel)
```javascript
export default async (req, res) => {
  const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);
  
  const session = await stripe.checkout.sessions.create({
    payment_method_types: ['card'],
    line_items: [{
      price_data: {
        currency: 'usd',
        product_data: {
          name: 'Boundary Buddy Premium',
          description: '1 month subscription',
        },
        unit_amount: 499,
        recurring: { interval: 'month' },
      },
      quantity: 1,
    }],
    mode: 'subscription',
    success_url: 'https://boundary-buddy.app?success=true',
    cancel_url: 'https://boundary-buddy.app',
  });

  res.json({ sessionId: session.id });
};
```

**Update upgradePremium():**
```javascript
async function upgradePremium() {
  const stripe = Stripe('pk_live_YOUR_PUBLIC_KEY');
  const response = await fetch('/api/stripe-checkout', { method: 'POST' });
  const { sessionId } = await response.json();
  stripe.redirectToCheckout({ sessionId });
}
```

### 4.4 Setup Environment Variables

**For Vercel:**
```
Project Settings → Environment Variables
Add:
- STRIPE_SECRET_KEY = sk_live_...
- STRIPE_PUBLIC_KEY = pk_live_...
```

**For Netlify:**
```
Site settings → Build & deploy → Environment
Add:
- STRIPE_SECRET_KEY
- STRIPE_PUBLIC_KEY
```

---

## Step 5: Setup Google Analytics

### 5.1 Create Analytics Account
```
1. analytics.google.com
2. Sign in with Google account
3. Create new property
4. Name: "Boundary Buddy"
5. Get tracking code (G-XXXXX)
```

### 5.2 Add to HTML

**Add to index.html `<head>`:**
```html
<!-- Google Analytics -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXX"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'G-XXXXX');
</script>
```

Replace `G-XXXXX` with your actual ID.

### 5.3 Custom Events

Add tracking for important events:

```javascript
// Track premium clicks
document.querySelector('.upgrade-btn').addEventListener('click', () => {
  gtag('event', 'upgrade_click');
});

// Track social shares
function shareTwitter() {
  gtag('event', 'share', { platform: 'twitter' });
  // ... rest of function
}

// Track referrals
function copyReferralCode() {
  gtag('event', 'referral_copy');
  // ... rest of function
}
```

---

## Step 6: Setup Email Service

### 6.1 Mailchimp Setup (Free)

```
1. mailchimp.com → Sign up
2. Create audience
3. Get API key (Account → Extras → API Keys)
4. Create email template
5. Setup double opt-in
```

**Add email signup form to website:**
```html
<!-- Mailchimp Signup -->
<form id="emailForm">
  <input type="email" placeholder="Enter email" required>
  <button type="submit">Subscribe</button>
</form>

<script>
document.getElementById('emailForm').addEventListener('submit', async (e) => {
  e.preventDefault();
  const email = e.target.querySelector('input[type="email"]').value;
  
  const response = await fetch('/api/subscribe', {
    method: 'POST',
    body: JSON.stringify({ email })
  });
  
  alert('Check your email to confirm!');
});
</script>
```

**Backend function (/api/subscribe.js):**
```javascript
const fetch = require('node-fetch');

export default async (req, res) => {
  const { email } = JSON.parse(req.body);
  
  const response = await fetch('https://us1.api.mailchimp.com/3.0/lists/YOUR_AUDIENCE_ID/members', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${process.env.MAILCHIMP_API_KEY}`
    },
    body: JSON.stringify({
      email_address: email,
      status: 'subscribed',
    })
  });
  
  res.json({ success: true });
};
```

---

## Step 7: Auto-Deployments

### 7.1 Push Updates Automatically Deploy

```bash
# Make a change to index.html
# Commit and push
git add index.html
git commit -m "Update feature"
git push origin main

# Vercel/Netlify automatically:
# 1. Detects push to GitHub
# 2. Runs build
# 3. Deploys live
# Done! 🚀
```

### 7.2 Check Deployment Status

**Vercel:**
```
Vercel Dashboard → Project → Deployments
Shows each deployment with timestamp and status
```

**Netlify:**
```
Site → Deploys → Shows deployment history
```

---

## Step 8: Testing Checklist

### Before Going Live

```
[ ] All links work (test on phone too)
[ ] Buttons respond instantly
[ ] Responsive on mobile (test on iPhone/Android)
[ ] Local storage works (open DevTools → Application → Local Storage)
[ ] Streak tracking works
[ ] Category selection works
[ ] Notes save properly
[ ] Social share buttons work
[ ] Premium upgrade button works (redirects)
[ ] Analytics tracking fires
[ ] Email signup works
[ ] Page loads in <3 seconds
[ ] No console errors (DevTools → Console)
```

### Test on Real Devices

```
Desktop: Chrome, Firefox, Safari
Mobile: iPhone (Safari), Android (Chrome)
Tablet: Test in landscape mode
```

---

## Step 9: Production Checklist

Before announcing:

```
[ ] Domain purchased and connected
[ ] SSL certificate active (https://)
[ ] Stripe live keys (not test keys)
[ ] Email service configured
[ ] Analytics tracking
[ ] Error monitoring (Sentry)
[ ] Custom domain working
[ ] All pages load quickly
[ ] Mobile experience perfect
[ ] Legal pages live (privacy, terms)
[ ] Support email working
[ ] Backup plan in place
```

---

## Troubleshooting

### Site Not Showing Up
```
1. Check DNS propagation: nslookup boundary-buddy.app
2. Wait 5-30 minutes for DNS to propagate
3. Clear browser cache (Cmd+Shift+R)
4. Try incognito mode
```

### Payment Not Working
```
1. Check Stripe keys are correct
2. Verify Stripe account is active
3. Test with Stripe test card: 4242 4242 4242 4242
4. Check browser console for errors (DevTools)
```

### Analytics Not Recording
```
1. Verify G- ID is correct
2. Check if analytics.js loaded (DevTools → Network)
3. Wait 24 hours for first data
4. Use Real-time tab in Analytics
```

---

## Monitoring (After Launch)

### Daily Checks
```
1. Site loads correctly
2. New user signups coming in
3. Check Stripe for charges
4. Monitor Twitter/Reddit mentions
5. Check email inbox for signups
```

### Weekly Checks
```
1. Analytics metrics
2. User feedback
3. Performance (Core Web Vitals)
4. Error logs (Sentry)
5. Email list growth
```

### Monthly Checks
```
1. User retention rates
2. Premium conversion rate
3. Referral performance
4. Revenue
5. Growth projections
```

---

**Ready to deploy? Start with Step 1! 🚀**
