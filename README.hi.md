# `@babtab/relay`

[English](README.md) | [繁體中文](README.zh-TW.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [Português (BR)](README.pt-BR.md) | [हिन्दी](README.hi.md)

[![संस्करण](https://img.shields.io/npm/v/@babtab/relay.svg)](https://www.npmjs.com/package/@babtab/relay)
[![लाइसेंस](https://img.shields.io/badge/license-Apache--2.0-green.svg)](LICENSE)

Babtab Chrome एक्सटेंशन के लिए **लोकल relay सर्वर**: यह आपके AI एजेंट (Cursor / Pi / Claude Code / …) को आपके असली Chrome से जोड़ता है।

MV3 एक्सटेंशन स्वयं किसी पोर्ट पर listen नहीं कर सकते, इसलिए यह छोटा प्रोग्राम bridge का काम करता है। यह `localhost` पर चलता है, इसलिए ट्रैफ़िक आपकी मशीन से बाहर कभी नहीं जाता। Relay सिर्फ़ संदेश आगे बढ़ाता है — **यह आपकी पेज सामग्री नहीं देख सकता**।

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
2. Relay URL में पहले से `ws://127.0.0.1:3000` भरा है — वैसे ही रहने दें
3. "Save & Connect" पर क्लिक करें

इससे आपका Chrome relay पर एक device के रूप में पंजीकृत हो जाता है।

### चरण 4: AI एजेंट को relay से जोड़ें (Cursor उदाहरण सहित)

1. Side Panel के उसी पेज पर अपना एजेंट चुनें → "Copy config" पर क्लिक करें
2. `~/.cursor/mcp.json` के `mcpServers` में paste करें (सिर्फ़ एक प्रोजेक्ट हेतु हो तो उस प्रोजेक्ट की `.cursor/mcp.json` में):

```json
{
  "mcpServers": {
    "babtab": {
      "url": "http://127.0.0.1:3000/mcp",
      "headers": { "Authorization": "Bearer अभी-मिला-टोकन" }
    }
  }
}
```

3. जब पैनल में `Controlled by: cursor` दिखे, तो कनेक्शन हो गया।

सत्यापन हेतु एक वाक्य (अपने एजेंट से कहें):

> browser_observe से मेरे Chrome में खुले टैब देखो, फिर उनके title और URL बताओ।

## अक्सर पूछे जाने वाले प्रश्न

- **"Save & Connect" दबाने पर कुछ नहीं होता?** पहले relay टर्मिनल में `listening` देखें, फिर पुष्टि करें कि URL `ws://127.0.0.1:3000` है और पोर्ट मेल खाता है।
- **पेयरिंग कोड समाप्त हो गया?** कोड थोड़ी देर ही चलते हैं — "Copy config" दोबारा दबाएँ।
- **नया टोकन चाहिए?** चरण 4 दोहराने पर नया टोकन मिलता है; `mcp.json` अपडेट करना न भूलें।
- **एजेंट और Chrome अलग मशीनों पर हैं?** (उन्नत) Relay को VPS पर रखें और Side Panel की Relay URL को अपने `wss://…` पर बदल दें। प्रक्रिया वही रहती है।

## गोपनीयता

- `localhost` पर यह पूरी तरह लोकल कनेक्शन है — पैकेट आपके कंप्यूटर से बाहर कभी नहीं जाते।
- Relay सिर्फ़ कमांड और परिणाम आगे बढ़ाता है। यह पेज सामग्री को कभी parse या store नहीं करता।
- Side Panel कभी भी **Pause / Take Over / Disconnect** कर सकता है — अंतिम नियंत्रण हमेशा इंसान के पास रहता है।

## डेवलपर्स

इस repo में सिर्फ़ release artifacts हैं (एक obfuscated bundle + binaries), development source नहीं। Issues और चर्चाएँ यहीं खोलें।

## लाइसेंस

Apache-2.0 — [LICENSE](LICENSE) और [NOTICE](NOTICE) देखें।

Copyright 2026 Poseidoncode.
