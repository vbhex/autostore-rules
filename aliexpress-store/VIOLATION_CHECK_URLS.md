# AliExpress Violation Check URLs

Check these links regularly to catch punishment notifications and audit failures early.

---

## Violation Punishment Notifications

**URL**: https://csp.aliexpress.com/m_apps/message/notice?channelId=2087779

**What to do**: Click each notification → read the violation detail → identify infringing brand → add to `banned-brands.json` in both `1688_scrapper/` and `aliexpress/` repos.

---

## My Violations (All / Pending Appeal / Active)

**URL**: https://csp.aliexpress.com/m_apps/violation/pop-violist?channelId=2087779

Filters available:
- **Pending Appeal** — product still live, appeal window open (usually 14 days)
- **Active** — product delisted/restricted, past appeal window

**Violation detail URL pattern**:
```
https://csp.aliexpress.com/m_apps/violation/pop-viodetail?punishId={ID}&appealChannel=CCO&channelId=2087779
```

---

## Account Health Dashboard

**URL**: https://csp.aliexpress.com/m_apps/account-health/pop-accounthealth?channelId=2087779

---

## Audit Failures (Review Failed Products)

**URL**: https://aeop.aliexpress.com/product/manage.htm?filterStatus=AUDIT_FAILED

Or navigate: Seller Center → Products → Product Management → filter "Not Approved"

---

## Brands Found in Violations (2026-03-10)

All brands below have been added to `banned-brands.json` in both repos:

| Brand | Violation Type | Notes |
|-------|---------------|-------|
| Mardi Mercredi | Trademark | Korean fashion brand — multiple products (daisy/dachshund sweatshirts) |
| Acne Studios | Trademark | "acne1996" in title triggers this |
| Maison Kitsune | Trademark | "MK double fox head" reference |
| Sanrio | Copyright | Characters: Hello Kitty, Cinnamoroll, Kuromi, etc. |
| Ralph Lauren | Trademark | "RL Ralph Pony Logo" |
| MIU MIU | Trademark | Girls' undershirt |
| Thom Browne | Trademark | Four stripes (similar to their signature design) |
| Alo Yoga / Aloyoga | Trademark | 3D styling hoodie |
| NASA | Trademark | Tie-dye pants; multiple products |
| TENCEL | Trademark | Lenzing fiber brand — do NOT use "TENCEL" as a material descriptor |
| Dior | Trademark | "Dior Men's..." in product title |
| Fear of God / FOG Essentials | Trademark | Obscured logo ESSENTIALS hoodie |
| 遮挡或涂抹 | Logo obscured | AliExpress flags products where supplier covered a brand logo in image |
