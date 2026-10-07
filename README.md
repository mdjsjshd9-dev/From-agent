# WhatsApp Boss Agent v3.0

Boss AI agent + 100+ employee agents, ek hi WhatsApp number se. Baileys + Gemini + SQLite. Sab free.

## Kya karta hai
- **Boss:** owner (CEO) ke message sunta hai aur tools se kaam karta hai.
- **Employees:** boss khud banata/nikalta hai (`hire_employees`, `fire_employees`, `update_employee`). Max 120 (`MAX_EMPLOYEES`). Har employee ka naam, role, persona, skills aur apna kaam hota hai.
- **Dispatcher:** naya contact aaye to Gemini role dekhkar sahi employee chunta hai. Contact usi employee ke paas rehta hai (sticky), boss `reassign_contact` se badal sakta hai.
- **Memory:** har contact ke notes SQLite me. Har ~16 messages ke baad notes khud update hote hain.
- **Natural style:** typing delay, chhote bubbles, markdown nahi, kai messages aayein to ek hi jawab.
- **Honesty rule (code me fixed):** koi sachmuch pooche ki insaan ho ya AI, employee jhooth nahi bolta. Wo batata hai ki wo company ka AI assistant hai aur team ke insaan tak baat pahuncha sakta hai.
- **Report:** roz `REPORT_HOUR` baje daily report owner ke chat me. Urgent escalation turant aati hai. Boss se "report de" bolo to abhi bhi mil jaata hai.
- **Approval:** hire, fire, broadcast, group se member hatana, group chhodna, block. Owner `YES P1` / `NO P1` likhe ya panel se Approve/Reject kare. 30 min me jawab na aaye to cancel. `HIRE_NEEDS_APPROVAL=false` karke hire ko free kar sakte ho, fire hamesha approval se.
- **Contact rules:** VIP, bot-level block, `QUIET_HOURS` (VIP ko chhod kar reply band), rate limit `REPLY_RATE_PER_HOUR`.
- **Pairing code:** link hone tak har ~45 sec me naya code, web panel + Telegram pe. Logout ho to session delete hokar naya code.

## Files
```
index.js        <- poora code ek hi file me (sections "=====" se alag)
package.json  Dockerfile  railway.json  .env.example  .gitignore  README.md
```
index.js ke andar sections: CONFIG, UTILS, DATABASE, WHATSAPP (send, pairing, router), GEMINI, EMPLOYEE AGENT, REPORTS, SCHEDULER, BOSS TOOLS + APPROVALS, BOSS AGENT, WEB PANEL.

## Phase 2: Voice, image, PDF
- **Voice note sunna:** contact ya owner voice note bheje to Gemini use transcribe karta hai. Owner ka voice note seedha command ban jata hai ("ek sales wala hire kar do").
- **Image dekhna:** photo ka chhota description aur usme likha zaroori text. ID/card number/OTP jaisi cheezein copy nahi hoti, sirf type batata hai.
- **PDF:** summary aur zaroori details. Doosre documents sirf file naam ke saath tag hote hain.
- **Safety:** media ke andar likhe instructions sirf data hain, follow nahi hote. Limits: `MAX_AUDIO_SEC`, `MAX_MEDIA_MB`, `MEDIA_PER_HOUR`. Fail ya limit par employee politely text me likhne ko bolta hai.
- **Voice ya text, insaan ki marzi se:** `VOICE_REPLY=true` ho to `VOICE_MODE=mirror` me voice note par voice aur text par text jawab aata hai. `VOICE_MODE=always` me chhote jawab hamesha voice me. Insaan "voice me batao" bole to us chat me voice chalu rehta hai, "text me likho" bole to wapas text (har chat ki pasand yaad rehti hai).
- **Gaana:** bot gaa nahi sakta, sirf bol sakta hai (Gemini TTS bolne wali awaaz hai). Mashhoor gaano ke bol copy nahi karta. Gaane ko bolo to wo apni chhoti original tukbandi/shayari bol sakta hai ya gaane ke baare me baat karta hai.
- Voice se pehle emoji aur links hata diye jate hain. Roman Hinglish ka uchcharan kabhi kabhi thoda alag aa sakta hai, `VOICE_NAME` badalke dekho.
- **Voice reply (optional):** `VOICE_REPLY=true` karo to voice note ke jawab me voice note jata hai (Gemini TTS + ffmpeg). Text message ka jawab text me hi rahta hai. TTS ya ffmpeg fail ho to text me chala jata hai. Dockerfile me ffmpeg pehle se hai.
- **Key rotation aur daily report** Phase 1 me hi aa chuke hain (kai `GEMINI_API_KEY` comma se, aur roz `REPORT_HOUR` pe report).
- Employee ab bhi sachmuch poochne par AI hone se inkaar nahi karta, voice me bhi.

## Phase 3: Groups + Security

### Groups
Default me bot kisi group me kuch nahi karta (`GROUPS_DEFAULT_MODE=off`). Boss chat se har group alag set hota hai, jaise:
- `Family group me bot ko mention par jawab dene wala bana de` (mode=mention)
- `Sales group me welcome message lagao: Welcome {name}, rules pin hain`
- `Sales group me anti-spam aur anti-link on kar`
- `Sales group ko Neha (E003) sambhale`
- `Sales group 30 min ke liye mute kar`

| Feature | Kaise kaam karta hai |
|---|---|
| Mode `mention` | Sirf tag karne ya bot ke message ka reply karne par jawab, sender ko @mention karke |
| Mode `always` | Har message par jawab (rate limit ke saath, sirf zaroorat par) |
| Welcome | Naye member par `{name}` ko mention karke text |
| Anti-spam | `GROUP_FLOOD_MSGS` messages `GROUP_FLOOD_SECS` sec me = warning. 3 warning ke baad **hatane ki approval aapse maangta hai**, khud nahi hatata |
| Anti-link | Non-admin ka link wala message delete (bot ko group admin hona zaroori) |
| Admins | Group admins aur owner par anti-spam/link lagu nahi |
| History | `mention`/`always` groups ki chat save hoti hai, boss se summary maang sakte ho |

### Security
- **Approval PIN** (`APPROVAL_PIN`): approve ke liye `YES P1 <PIN>`. 3 galat PIN = 15 min lock + alert. Panel me bhi PIN poochta hai. `NO` ke liye PIN nahi chahiye.
- **Prompt-injection guard:** "ignore instructions", "system prompt dikha" jaise messages pehchaan kar employee ko savdhan kiya jata hai aur event log hota hai. Yeh andaaza lagane wala filter hai, 100% nahi. Asli suraksha yeh hai ki employees ke paas bahut limited tools hain aur khatarnak kaam approval se hote hain.
- **Reply scrubber:** employee ka jawab bhejne se pehle check hota hai. API key, password, PIN, system prompt, ya owner ka number (agar `COMPANY_INFO` me business number ke roop me nahi hai) nikle to jawab roka jata hai, sender ko neutral jawab jata hai, aur aapko alert.
- **Flood guard:** `GLOBAL_INBOUND_PER_MIN` se zyada messages/min aaye to auto-reply 10 min ruk jata hai (quota aur ban se bachav).
- **Data retention:** `RETENTION_DAYS` se purani chats roz auto-delete. Contact notes rehte hain.
- **Forget contact:** `Rahul ka saara data delete kar` bolo, approval ke baad chat, notes, tasks, scheduled messages sab hat jate hain.
- **Alerts:** WhatsApp `DOWN_ALERT_MIN` se zyada disconnected rahe, ya Gemini lagatar fail ho, to Telegram/owner ko alert. Wapas online hone par bhi.
- **Panel:** Groups tab aur Approval PIN status add hue. Pehle wali sab suraksha (lockout, CSP, HttpOnly cookie) waisi hi hai.

### Zaroori baatein
- Anti-link delete aur member remove ke liye bot ko us group me **admin** banana padega.
- Group me employee "AI hai ya insaan" poochne par ab bhi jhooth nahi bolta.
- Group ke members ki privacy ke liye bot kisi ki private jankari doosron ko nahi deta.

## History, chat aur memory (owner chat se)
Commands `!` ya `/` dono se chalte hain. **Time** kahin bhi likho: `15min`, `2h`, `2day`, `1week`, `2month`, `5month`, `all`.

| Command | Kya dikhata hai |
|---|---|
| `!history` (default 1 din) | Boss aur employees ne kya kiya: messages ginti, har employee ke kitne jawab, aur actions/events ki time-wise list (hire, fire, approvals, messages bheje, calls, spam, escalations) |
| `!history 15min` / `2day` / `5month` / `all` | Wahi, chuni hui time ke liye |
| `!chat 919999999999 2day` | Us number ki profile (naam, employee, notes, tasks) aur us time ki poori chat. Time na do to sab. Naam bhi chalta hai: `!chat Rahul 15min` |
| `!memory 919999999999` | Sirf profile: us insaan ke baare me jo yaad hai |
| `!forget 919999999999` | Us insaan ka saara data delete, **approval ke baad** (PIN laga ho to PIN ke saath) |

Lambi chat chhote hisso me aati hai (aakhri 100 messages dikhte hain, chhota time do to poori). "Peeche kitna dikhega" `RETENTION_DAYS` (default 180 din) par depend karta hai; usse purani chats auto-delete hoti hain. Audit log kam se kam 365 din rehta hai.

### Dheere dheere insaan ko jaanna
- Employee chat me natural tareeke se naam, kaam, zaroorat, language/time pasand jaise business ke kaam ke details seekhta hai, **sirf wahi jo insaan khud bata de**. Bahut poochh-taachh nahi, ek baar me ek halka sawal.
- Naya zaroori pata chale to `save_note` se profile me likhta hai, aur har 10 naye messages par poori memory khud update hoti hai.
- Bot kisi number ki bahar se jaanch-padtaal nahi karta (koi lookup, tracking ya search nahi). Sirf chat me jo bataya gaya.
- Sensitive cheezein (Aadhaar/PAN/ID, card/bank details, password/OTP, health, private baatein) na poochhta hai na note me likhta hai.

### Memory kaun hata sakta hai
- **Sirf owner.** Employees ke paas koi delete tool hi nahi hai. Koi insaan "mera data delete karo" bole to employee `notify_boss` karke aapko batata hai, aur aap `!forget` se faisla karte ho.
- Contact ke kehne par ya chat ke andar likhe kisi instruction se boss kuch delete nahi karta.
- Delete hamesha approval se hota hai. Alag se, `RETENTION_DAYS` se purani chats automatic hat jati hain (contact ke notes rehte hain).
- Employees ke har tool use (note save, task banana, boss ko batana) ka record audit me jata hai, jo `!history` me dikhta hai.

## Setup
1. Gemini key(s): https://aistudio.google.com/apikey (kai keys comma se alag, limit lagne par agli key chalti hai).
2. GitHub repo me poora folder push karo.
3. Railway: Deploy from GitHub. **Volume add karo, mount path `/data`.**
4. Variables: `.env.example` dekho. Zaroori: `OWNER_NUMBER`, `GEMINI_API_KEY`, `ADMIN_PASSWORD`, `DATA_DIR=/data`.
5. Settings > Networking > **Generate Domain**. Link kholo, password daalo, pairing code panel me dikhega.
6. WhatsApp > Linked devices > Link with phone number > code daalo.

Node 22.13+ chahiye (built-in `node:sqlite` use hota hai, koi native build nahi).

## Business customize kaise karein (variables se)
Code ek hi rehta hai, sirf Railway variables badalke alag project bana lo (portfolio demo, dost ka business, papa ka kaam). Variable badalne ke baad Railway redeploy ho jata hai.

| Variable | Matlab |
|---|---|
| `COMPANY_NAME` | Business ka naam |
| `COMPANY_INFO` | Services, prices, timings, address, policies, FAQs. Ek line me likhna ho to `\n` lagao (Railway me multi-line bhi chalta hai) |
| `DEFAULT_EMPLOYEE_NAME/ROLE/PERSONA` | Pehla employee, jo pehli baar DB banne par ban jata hai |
| `BOT_NAME` | Panel aur boss ka naam |

Variable set hai to wahi chalega. Khali ho to boss chat se `set_company_info` bolke bhi set kar sakte ho. Naya business hai to alag Railway project aur alag Volume lo, taaki chats aur session mix na hon.

**Example 1: Student portfolio demo**
```
COMPANY_NAME=Aarav Web Studio
COMPANY_INFO=Hum websites aur chatbots banate hain.\nPrice: landing page 3000 se, portfolio site 5000 se.\nTiming: Mon-Sat 10am-7pm.\nDemo ke liye "demo" likho, aur kaam ke liye apna budget aur deadline batao.
DEFAULT_EMPLOYEE_NAME=Riya
DEFAULT_EMPLOYEE_ROLE=Client Support
```

**Example 2: Kisi shop/kaam ke liye**
```
COMPANY_NAME=Sharma Hardware
COMPANY_INFO=Paint, pipes, electricals ka saaman.\nTiming: 9am-8pm, Sunday band.\nHome delivery 5 km tak free, 1000 se upar ke order par.\nRates badalte rehte hain, isliye final price ke liye owner se puchhna hoga.
DEFAULT_EMPLOYEE_NAME=Raju
DEFAULT_EMPLOYEE_ROLE=Counter Staff
DEFAULT_EMPLOYEE_PERSONA=Seedhi, simple Hindi me baat karta hai.
```
Jo cheez COMPANY_INFO me nahi hai, employee use banata nahi; wo `notify_boss` se aapko bata deta hai.

## v3.0: naye features

### Hamesha ON (koi variable nahi chahiye)
- **Web search:** boss aur employees internet se taaza jankari dekh sakte hain (Gemini ka Google Search tool, isi key se). Employee ke liye 5 search/ghanta per chat. Web ka text "untrusted data" maana jata hai.
- **Lead scoring:** har contact ki chat se Gemini stage (`new, interested, hot, customer, cold, not_interested`), score (0-100), kyun aur agla kadam nikalta hai. Hot lead par aapko turant alert. `!leads`, panel ka Leads tab, daily report me ginti.
- **Follow-up:** interested/hot lead ne jawab dena band kiya to employee halka, polite follow-up bhejta hai. **Safeguards:** sirf unko jinhone khud message kiya tha (pichle 7 din me), max 2 baar per lead, 48 ghante ka gap, sirf din ke samay (10-20), roz max 10, "stop / message mat karo" bolne wale ko kabhi nahi, quiet hours aur auto-reply off me nahi. `!followups off` se band.
- **Media library:** image/PDF bhejo, caption me `! save poster` likho. Employees "poster bhej do" jaise maange par `send_media` se bhejte hain. `!media` se list.
- **PDF banakar bhejna:** quotation, invoice, summary (`send_pdf`). Koi library nahi lagti. Sirf Latin text (Hindi akshar `?` ban jate hain), `Rs.` likho. Employee sirf wahi price/terms likhta hai jo BUSINESS INFO ya boss ke order me hain.
- **Image bhejna:** public https link se (`send_image_url`, private/local address aur redirect band) ya AI se banakar (`generate_image`). **Gemini image generation sabhi free keys par nahi milta**, na mile to bot saaf bata deta hai.

### Variable se ON (`!features` me dikhta hai kya ON/OFF hai)
| Feature | Railway variable | Kya milta hai |
|---|---|---|
| Knowledge base | `KB_ENABLED=true` (optional `KB_URLS`) | Employees documents se jawab dete hain. File bhejo, caption `! kb naam` (PDF, txt, md, csv, image). Gemini embeddings, SQLite me store |
| Backup | `BACKUP_TELEGRAM=true` **ya** `BACKUP_S3_ENDPOINT/BUCKET/KEY/SECRET` | Roz database backup (gzip, `BACKUP_PASSPHRASE` ho to AES-256 encrypted). `!backup now` se abhi. Restore: `node index.js restore <file>` |
| Google Sheets | `GOOGLE_SERVICE_ACCOUNT_JSON` + `GOOGLE_SHEET_ID` | Leads aur daily report sheet me khud likhe jate hain; boss `sheet_append/sheet_read` |
| Google Calendar | `GOOGLE_SERVICE_ACCOUNT_JSON` + `GOOGLE_CALENDAR_ID` | Employees khali slot dekhkar meeting book karte hain (kaam ke ghanto me), aapko alert. Cancel aapki approval se |
| Webhook API | `WEBHOOK_TOKEN` | `POST /hook/message`, `/hook/lead`, `/hook/boss` (boss ke liye `WEBHOOK_ALLOW_BOSS=true` bhi) |

Variable na lage to wo feature ke tools, commands aur prompt me mention hi nahi aate.

**Backup ke baare me:** Cloudflare R2 (10 GB free) ya Backblaze B2 ya AWS S3 chalta hai. Telegram me 50 MB tak. WhatsApp ka login (auth folder) backup me nahi jata (suraksha ke liye); Volume udd jaye to bas dobara pairing code se link karna padega. Google Drive support nahi hai kyunki service account personal Drive me file nahi bana sakta.

**Google setup:** Google Cloud me project > Service Account banao > JSON key download karo > poori JSON `GOOGLE_SERVICE_ACCOUNT_JSON` me daalo. Sheets/Calendar API enable karo. Phir sheet aur calendar ko service account ke email se "Editor" share karo. Sheet ID URL me hota hai; Calendar ID calendar ki settings me.

**Webhook example:**
```
curl -X POST https://<aapka-domain>/hook/lead \
  -H "Authorization: Bearer <WEBHOOK_TOKEN>" -H "Content-Type: application/json" \
  -d '{"name":"Rahul","number":"919876543210","message":"Website banwani hai","source":"website","greet":true}'
```
`greet:true` par employee us insaan ko warm greeting bhejta hai (sirf tab karo jab form me message ki ijazat li ho). Limit 60 requests/minute.

### Naye commands
`!leads`, `!followups on|off`, `!media`, `!kb`, `!backup [now]`, `!features`. Ye bhi bolke chalte hain: "! internet se dollar rate dekho", "! hot leads dikhao", "! Rahul ko quotation PDF bhejo: logo 3000".

### Dhyan rakho
- Lead scoring aur follow-up me extra Gemini calls lagte hain (free limit). Follow-up ka message customer ko jata hai, isliye shuru me `!followups off` karke chhote test se dekh lo.
- Follow-up unofficial WhatsApp (Baileys) par unsolicited-jaisa message hai, ban ka risk badh sakta hai. Limits isiliye rakhe hain.
- KB aur backup me aapke customers ka data Google (embeddings) ya aapke chune hue storage me jata hai.

## Boss se seedhi baat (jaise AI se chat karte ho)
Apne "Message yourself" chat me `!` ya `/` ke baad **kuch bhi apni bhasha me** likho. Boss AI hai, poora system uske haath me hai, wo data dekhkar jawab deta hai aur kaam karta/karwata hai:

```
! aaj kitne naye logon se baat hui
! abhi tak tumne kya kiya, last 15 min
/ Rahul ne kya kaha, aur kisne price poocha
! Riya ka aaj ka kaam dikha
/ Neha se bolo kal 5 baje Amit ko meeting yaad dilaye
! sab employees ko order do: discount kabhi mat dena, mujhse puchho
/ sales group me kya chal raha hai, summary do
! 3 support employees hire kar
! system ka haal batao
```
- **Secret/trigger:** self-chat me jo message `!` ya `/` se shuru nahi hota wo boss ko nahi jata (aapke personal notes safe). Apna secret chahiye to `BOSS_TRIGGER=.b,!` jaisa set karo. `YES P1` / `NO P1` aur voice note bina trigger ke chalte hain.
- **Reply ke aage 🤖** aata hai, taaki aapke apne notes se alag dikhe. Alag bot number use karo to trigger zaroori nahi (`OWNER_NEEDS_TRIGGER=false` apne aap).
- **Yaad rehta hai:** boss ki baatcheet DB me save hoti hai, restart ke baad bhi "pichli baat" yaad rehti hai.
- **Boss ke naye tools:** `new_contacts`, `activity_summary`, `search_messages`, `employee_activity`, `system_status`, `chat_history (time ke saath)`, `owner_command`.
- **Employees se kaam karwana (`delegate_work`):** boss sahi role ka employee chunta hai, employee apni persona me contact ko message likhke bhejta hai (honesty rule ke saath). "to" na do to employee ke liye task ban jata hai. 3 se zyada logon ko bhejna ho to aapki approval lagti hai.
- **Standing orders (`add_order`):** hamesha ke niyam, sab employees ke liye ya kisi ek ke liye ("discount mat dena", "Neha sirf support dekhe"). Har employee ke prompt me judte hain. Jo order employee ko "insaan hone ka dawa" ya "AI hone se inkaar" sikhaye wo accept nahi hota.
- **Boss se poochh sakte ho ki use kaise karein:** `! bhai tum kaise use karun`, `/ tum kya kya kar sakte ho`, `! example do`. Boss poora guide (kya poochh sakte ho, kya karwa sakte ho, approval, commands, kya nahi kar sakta) apni bhasha me aur aapke system ke hisaab se samjhata hai.
- **Fixed commands bhi chalte hain** (`!status`, `!history 2day`, `!chat 9199... 5day`...), jo instant hain aur AI quota nahi lete.

### Dhyan rakho
- Boss ke har sawal par 1-4 Gemini calls lagte hain. Free limit me hi rehna ho to simple cheezein `!command` se karo.
- Self-chat me voice note ko command mana jata hai (transcribe hone ke liye Gemini ko jata hai). Personal voice memo wahan mat bhejo, ya `OWNER_VOICE=false` karo.
- Boss kuch bhi delete tabhi karta hai jab aap approve karo. Employees ya contacts ke kehne par kabhi nahi.

## Owner commands (!help)
Owner chat (ya "Message yourself") me `!help` likho to menu aata hai. `!commands` instant hain, AI quota nahi lagta:

| Command | Kaam |
|---|---|
| `!help` | Menu aur examples |
| `!status` | WhatsApp, employees, tasks, approvals, auto-reply, uptime |
| `!employees` / `!employees all` | Employees ki list |
| `!tasks` / `!tasks done` / `!tasks all` | Tasks |
| `!pending` | Approval ke wait me kaam |
| `!report` | Abhi ka report |
| `!groups` | Groups ke settings |
| `!calls` / `!callmode ...` | Haal ki calls / call mode badlo |
| `!history`, `!chat`, `!memory`, `!forget` | Neeche "History, chat aur memory" dekho |
| `!contacts [naam]` | Recent contacts ya search |
| `!autoreply on/off` | Dusron ko auto-reply chalu/band |

Approval ke liye `YES P1` / `NO P1` (PIN laga ho to `YES P1 <PIN>`). `!` ke bina jo bhi likho (ya voice note bhejo), boss AI use samajhkar kaam karta hai: hire/fire, message bhejna, schedule, groups, contacts, company info, data delete, sab.

"Message yourself" tab chalta hai jab bot aapke apne number pe linked ho (`PAIRING_NUMBER` khali ya `OWNER_NUMBER` ke barabar). Us chat me notification na aaye to alag bot number behtar hai.

## WhatsApp calls
**Bot call utha ya baat nahi kar sakta.** WhatsApp ka yeh (unofficial Baileys) tareeka sirf call ki khabar deta hai aur call reject karne deta hai, audio sunne/bolne ka raasta nahi deta. Is liye bot calls ko *manage* karta hai:

| `CALL_MODE` | Kya hota hai |
|---|---|
| `off` | Kuch nahi |
| `notify` (default) | Call ka log (daily report me aata hai). VIP ki call par aapko turant alert |
| `message` | Upar wala + caller ko auto text ("call attend nahi ho paati, yahin message/voice note bhej dijiye"), 10 min me ek hi baar |
| `reject` | Upar wala + call reject. VIP ki call reject nahi hoti |

- Owner ki apni call aur group calls ignore hoti hain.
- Chat se badlo: `!callmode reject`, `!calls` (haal ki calls), ya bolo "calls reject kar do".
- Bot aapke personal number pe hai to `reject` mat rakhna, warna sabki call kat jayegi. `notify` ya `message` theek hain.
- Asli baat-cheet wala voice agent is setup se nahi banta. Uske liye official WhatsApp Business Calling (Meta business account aur approval) jaisa alag raasta chahiye, jo mere hisaab se abhi is project me nahi hai.

## Owner kaise use kare
Bot alag number pe ho to us number ko message karo. Apne hi number pe ho to "Message yourself" chat (notification na aa sake, isliye alag bot number behtar).

Examples:
- `company info set kar: naam Acme, timing 10-7, services web design aur SEO`
- `5 employees hire kar: 2 sales, 2 support, 1 scheduler`
- `E004 ko nikal de`
- `Rahul ko VIP bana de aur E002 ko de de`
- `aaj ki report de`, `pending approvals dikha`, `kal 6 baje Aman ko task do follow-up ka`

## Web panel
Status + pairing code, employees, contacts, tasks, approvals (Approve/Reject), reports, audit log. 5 galat password ke baad 10 min lock, CSP lagi hai, cookie HttpOnly.

## Limits (seedhi baat)
- Baileys unofficial hai, ban ka risk rehta hai. Isliye delays aur rate limits hain. 100 alag numbers mat chalao, employees ek number ke andar hi hain.
- Gemini free tier ki limits hain. Zyada traffic me jawab late ho sakte hain.
- Voice note ka jawab TTS voice me hota hai, aur Gemini TTS free tier ki limits hain. Groups me bot ko admin banana padta hai (anti-link/remove ke liye).
- Baileys version agar pehle project jaisa chahiye to `package.json` me wahi daalo.
- Code ka logic test kiya gaya (DB, tools, approvals, panel), par asli WhatsApp/Gemini ke saath pehli baar chalane par chhote bugs aa sakte hain. Railway logs bhej dena.
