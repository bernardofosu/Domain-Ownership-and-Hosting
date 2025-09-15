1) Make sure the BIMI DNS host is exactly
default._bimi.nakodtech.xyz


Run:

dig +short TXT default._bimi.nakodtech.xyz @1.1.1.1


You should see one TXT line starting with v=BIMI1;.

If the host were mistyped (e.g., default_bimi), or if multiple TXT records exist, BIMI will be ignored.

2) If you don’t have a VMC yet, remove the a= (or leave it empty)

Right now your record shows a=https://nakod…. That URL must point to a real VMC .pem. If you don’t have a VMC, keep it empty or omit it so non-Gmail providers can still use your BIMI:

TXT value (no VMC yet):

v=BIMI1; l=https://nakodtech.xyz/.well-known/bimi/logo.svg;


TXT value (with VMC):

v=BIMI1; l=https://nakodtech.xyz/.well-known/bimi/logo.svg; a=https://nakodtech.xyz/.well-known/bimi/vmc.pem


Then check again:

dig +short TXT default._bimi.nakodtech.xyz @1.1.1.1

3) Confirm your DMARC is at enforcement

You already set:

_dmarc.nakodtech.xyz TXT  v=DMARC1; p=quarantine; pct=100; …


That’s good. In a test message sent to Gmail, open ⋮ → Show original and confirm DMARC=pass and that it is aligned (the “From” domain is the same as the domain that passed SPF or DKIM).

4) Validate the SVG logo file

Your logo must be:

Hosted at https://nakodtech.xyz/.well-known/bimi/logo.svg

Square and formatted as SVG Tiny 1.2 (P/S) with no external fonts/links

Served with the correct MIME type

Check quickly:

curl -I https://nakodtech.xyz/.well-known/bimi/logo.svg


Expect HTTP/2 200 and content-type: image/svg+xml.

If you’re on Nginx, you can force the type:

types { image/svg+xml svg; }


Use a BIMI validator (search “BIMI inspector”) to confirm the SVG and DNS are valid.

5) Understand provider behavior

Gmail: requires VMC (+ DMARC enforcement) to show the round logo and the blue checkmark. Without a VMC, Gmail will keep showing the colored initial.

Yahoo/AOL/Fastmail: often display BIMI without a VMC once DMARC + SVG + DNS validate. Allow 24–72 hours and try sending to a Yahoo mailbox to test.

Quick fix checklist

 default._bimi.nakodtech.xyz exists (one TXT only)

 If no VMC: remove a= (or leave it empty)

 SVG reachable, image/svg+xml, valid BIMI SVG Tiny, square

 DMARC at enforcement and pass on real messages

 Test with Yahoo; for Gmail, plan to obtain a VMC (Entrust/DigiCert)

If you want, paste the outputs of:

dig +short TXT default._bimi.nakodtech.xyz @1.1.1.1
curl -I https://nakodtech.xyz/.well-known/bimi/logo.svg