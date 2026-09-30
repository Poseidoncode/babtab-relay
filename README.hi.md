# `@babtab/relay`

[English](README.md) | [繁體中文](README.zh-TW.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [Português (BR)](README.pt-BR.md) | [हिन्दी](README.hi.md)

[![संस्करण](https://img.shields.io/npm/v/@babtab/relay.svg)](https://www.npmjs.com/package/@babtab/relay)
[![लाइसेंस](https://img.shields.io/badge/license-Apache--2.0-green.svg)](LICENSE)

Babtab Chrome एक्सटेंशन के लिए **लोकल relay सर्वर**: यह आपके AI एजेंट (Cursor / Pi / Claude Code / …) को आपके असली Chrome से जोड़ता है।

MV3 एक्सटेंशन स्वयं किसी पोर्ट पर listen नहीं कर सकते, इसलिए यह छोटा प्रोग्राम bridge का काम करता है। यह `localhost` पर चलता है, इसलिए ट्रैफ़िक आपकी मशीन से बाहर कभी नहीं जाता। Relay अनुरोध और परिणाम सिर्फ़ मेमोरी में आगे बढ़ाता है (पेज अवलोकन और स्क्रीनशॉट सहित); यह पेज सामग्री को store नहीं करता।

## अनुशंसित सेटअप (0.2.0)

Node.js 20+ आवश्यक है। Chrome Web Store से एक्सटेंशन इंस्टॉल करें और Babtab में **Add to Cursor** दबाएँ। Cursor में जोड़ने और सक्षम करने की पुष्टि करें, फिर Chrome में लौटकर **Approve** दबाएँ। AI टूल Relay को अपने आप शुरू करता है; टर्मिनल खुला रखने की जरूरत नहीं है। अन्य टूल के लिए **Other AI tools / install with a command** चुनें।

बदलने से पहले पुराना मैन्युअल Relay बंद करें। [नवीनतम गाइड](README.md) देखें। नीचे दिए गए निर्देश मैन्युअल HTTP कनेक्शन के लिए हैं।

## प्रमाणित डिवाइस कनेक्शन पर अपग्रेड

Relay और एक्सटेंशन दोनों को एक साथ अपडेट करें, एक्सटेंशन reload करें, Side Panel में फिर से connect करें, और हर एजेंट के लिए चरण 2 दोहराएँ। एक्सटेंशन नई `dev_v2_` डिवाइस ID बनाता है; पुरानी ID से जुड़े टोकन नया कनेक्शन नियंत्रित नहीं कर सकते। Relay अब स्पष्ट रूप से सिर्फ़ `127.0.0.1` पर bind होता है। रिमोट डिप्लॉयमेंट में `HOST` सेट करें और ट्रांज़िट में क्रेडेंशियल की सुरक्षा के लिए TLS (`wss://` / `https://`) इस्तेमाल करें।

## यह कैसे जुड़ता है (3 भूमिकाएँ)

```text
AI एजेंट (Cursor / Pi …) ←→ Relay (लोकल :3000) ←→ Chrome एक्सटेंशन (Side Panel सहित)
   MCP कॉन्फ़िग से जुड़ता है    सिर्फ़ फ़ॉरवर्डिंग + पेयरिंग     आपके टैब में असली काम करता है
```

## त्वरित शुरुआत (clone नहीं, install नहीं)

### चरण 1: Chrome एक्सटेंशन इंस्टॉल करें

`chrome://extensions` → **Developer mode** चालू करें → **Load unpacked** → `dist` फ़ोल्डर चुनें।

> Chrome Web Store पर प्रकाशित होने के बाद यह चरण "स्टोर से इंस्टॉल करें" हो जाएगा।

### चरण 2: Relay शुरू करें (कोई एक चुनें, परिणाम समान)

```bash
# A. Node 20+ है? इंस्टॉल किए बिना सीधे चलाएँ:
npx @babtab/relay

# B. Node नहीं है? GitHub Releases से अपने OS की binary डाउनलोड करें:
./babtab-relay-darwin-arm64   # उदा. macOS Apple Silicon
```

जब यह दिखे तो समझें चल पड़ा (डिफ़ॉल्ट पोर्ट `3000`):

```text
[babtab-relay] listening on http://127.0.0.1:3000
```

उन्नत (कस्टम पोर्ट / टोकन फ़ाइल स्थान):

```bash
PORT=3001 npx @babtab/relay
BABTAB_TOKEN_FILE=~/.babtab/relay-tokens.json npx @babtab/relay
```

> `npx` कुछ इंस्टॉल नहीं करता — यह एक बार डाउनलोड करके चलाता है। टर्मिनल बंद करते ही relay रुक जाता है।

### चरण 3: एक्सटेंशन को relay से जोड़ें

1. Chrome में कोई भी वेबसाइट खोलें, Babtab टूलबार आइकन पर क्लिक करके **Side Panel** खोलें
2. चरण 1: पोर्ट की पुष्टि करें (डिफ़ॉल्ट `3000`, relay से मेल खाना चाहिए) — URL टाइप करने की ज़रूरत नहीं
3. "Save & Connect" पर क्लिक करें

इससे आपका Chrome relay पर एक device के रूप में पंजीकृत हो जाता है।
(VPS पर रिमोट relay? उसी चरण में *Advanced: custom Relay URL* इस्तेमाल करें।)

### चरण 4: AI एजेंट को relay से जोड़ें (Cursor उदाहरण सहित)

1. Side Panel के उसी पेज पर चरण 2 → **Cursor** पर क्लिक करें → एक-पंक्ति कमांड copy करें:

```bash
npx -y @babtab/relay setup --target cursor --relay-url http://127.0.0.1:3000 --device <आपकी-device-id>
```

2. किसी भी टर्मिनल में paste करके चलाएँ। 6-अंकीय पेयरिंग कोड दिखेगा —
   Side Panel के ऊपर banner में **Approve** करें।
3. हो गया: कमांड आपके लिए `~/.cursor/mcp.json` में `babtab` लिख देता है
   (मौजूदा entries सुरक्षित, पुरानी फ़ाइल `.bak` के रूप में रहती है)। Cursor में MCP
   reload करें (Settings → MCP) और पैनल में `Controlled by: cursor` दिखेगा।

समर्थित `--target` मान: `cursor`, `claude-code`, `windsurf`, `copilot`,
`copilot-insiders`, `codex`, `pi`, `claude-desktop`, `antigravity`, `devin`,
`kimi`, `hermes`, `manual`। सभी कमांड हेतु `npx -y @babtab/relay setup --help` चलाएँ
(`--list-targets` हर harness हेतु तैयार पंक्ति print करता है)। टर्मिनल उपलब्ध न हो?
चरण 2 में *Pair & copy JSON manually* विकल्प भी है।

3. जब पैनल में `Controlled by: cursor` दिखे, तो कनेक्शन हो गया।

सत्यापन हेतु एक वाक्य (अपने एजेंट से कहें):

> browser_observe से मेरे Chrome में खुले टैब देखो, फिर उनके title और URL बताओ।

## अक्सर पूछे जाने वाले प्रश्न

- **"Save & Connect" दबाने पर कुछ नहीं होता?** पहले relay टर्मिनल में `listening` देखें, फिर पुष्टि करें कि चरण 1 का पोर्ट relay के पोर्ट से मेल खाता है।
- **पेयरिंग कोड समाप्त हो गया?** कोड थोड़ी देर ही चलते हैं — setup कमांड दोबारा चलाएँ।
- **नया टोकन चाहिए?** setup कमांड दोहराने पर नया टोकन मिलता है और कॉन्फ़िग फिर से लिखी जाती है (पुरानी `.bak` बनी रहती है)।
- **एजेंट और Chrome अलग मशीनों पर हैं?** (उन्नत) Relay को VPS पर रखें, Side Panel चरण 1 में *Advanced: custom Relay URL* में अपना `wss://…` भरें और setup कमांड में `--relay-url https://…` जोड़ें। प्रक्रिया वही रहती है।

## गोपनीयता

- `localhost` पर यह पूरी तरह लोकल कनेक्शन है — पैकेट आपके कंप्यूटर से बाहर कभी नहीं जाते।
- Relay सिर्फ़ कमांड और परिणाम आगे बढ़ाता है। यह पेज सामग्री को कभी parse या store नहीं करता।
- Side Panel कभी भी **Pause / Take Over / Disconnect** कर सकता है — अंतिम नियंत्रण हमेशा इंसान के पास रहता है।

## डेवलपर्स

इस repo में सिर्फ़ release artifacts हैं (एक obfuscated bundle + binaries), development source नहीं। Issues और चर्चाएँ यहीं खोलें।
