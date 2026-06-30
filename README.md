# luhux-card

Headless Chrome kullanarak şık alıntı/avatar kartları (poster, sinema, minimal stilleri) PNG buffer olarak üretir.

## Kurulum

```bash
npm install luhux-card
```

Puppeteer bağımlılık olarak gelir, bu yüzden ilk kurulumda bir Chromium ikili dosyası indirilir. Çoğu ortamda ekstra bir ayar gerekmez.

## Hızlı başlangıç (tek seferlik render)

```ts
import { generateQuoteCard } from 'luhux-card';
import fs from 'fs';

const buffer = await generateQuoteCard({
    avatarUrl: 'https://cdn.discordapp.com/avatars/.../avatar.png',
    displayName: 'luhux1337',
    username: 'luhux',
    text: "selamun aleyküm :D",
    style: 'cinema',      // 'cinema' | 'poster' | 'minimal'
    accent: '#10b981',
    bg: '#0a0a0f',
    fontKey: 'inter',
    watermark: 'my-bot.dev',
});

fs.writeFileSync('quote.png', buffer);
```

## Tekrarlı render'lar (Discord butonları / editörler)

Aynı kartı birden fazla kez render alıyorsan (örneğin kullanıcı kaydetmeden önce butonlarla stil/renk değiştiriyorsa), `QuoteCardRenderer`'ı doğrudan kullan — her render'da Chromium'u yeniden başlatmak yerine tek bir browser+page'i açık tutar. Hız konusunda asıl fark burada.

```ts
import { QuoteCardRenderer } from 'luhux-card';

const renderer = new QuoteCardRenderer();

// ... collector içinde, kullanıcı her ayar değiştirdiğinde:
const buffer = await renderer.render(state);

// collector bittiğinde:
await renderer.close();
```

### Örnek: Discord.js slash command

```ts
import { QuoteCardRenderer, type QuoteCardOptions } from 'luhux-card';
import { AttachmentBuilder } from 'discord.js';

const renderer = new QuoteCardRenderer();

const state: QuoteCardOptions = {
    avatarUrl: user.displayAvatarURL({ size: 512, extension: 'png' }),
    displayName: user.globalName ?? user.username,
    username: user.username,
    text: interaction.options.getString('text', true),
    style: 'cinema',
};

const buf = await renderer.render(state);
await interaction.reply({ files: [new AttachmentBuilder(buf, { name: 'quote.png' })] });

// daha sonra, component collector bittiğinde:
await renderer.close();
```

## Seçenekler (`QuoteCardOptions`)

| Seçenek | Tip | Varsayılan | Not |
|---|---|---|---|
| `avatarUrl` | `string` | — | zorunlu |
| `displayName` | `string` | — | zorunlu |
| `username` | `string` | — | zorunlu, `@kullaniciadi` olarak gösterilir |
| `text` | `string` | — | zorunlu, alıntı metni |
| `style` | `'cinema' \| 'poster' \| 'minimal'` | `'cinema'` | |
| `fontKey` | `FONTS` içindeki bir anahtar | `'inter'` | aşağıya bak |
| `customFont` | `{ label, css, family }` | — | kendi Google Font'unu kullanmak için |
| `bg` | hex renk | `#0a0a0f` | arka plan rengi |
| `accent` | hex renk | `#10b981` | vurgu / gradient rengi |
| `tc` | hex renk | `#f0ece4` | metin rengi (poster ve minimal stilleri) |
| `fontSize` | `number` | `22` | temel boyut; sinema modu metin uzunluğuna göre buna göre otomatik ölçeklenir |
| `dim` | `number` (10–90) | `55` | fotoğrafın ne kadar karartılacağı |
| `watermark` | `string` | `'luhux-card'` | sinema stilinde sol altta gösterilen yazı |
| `width` / `height` | `number` | `920` / `460` | tuval boyutu |
| `scale` | `number` | `2` | ekran görüntüsü cihaz ölçek çarpanı (netlik / hız dengesi) |

## Hazır presetler

`BG_COLORS`, `ACCENT_COLORS`, `TEXT_COLORS` ve `FONTS` dışa aktarılır, böylece referans botla aynı şekilde kendi seçim menülerini (örneğin Discord select menu'leri) oluşturabilirsin.

```ts
import { FONTS, ACCENT_COLORS } from 'luhux-card';

Object.keys(FONTS);          // 'cormorant' | 'playfair' | 'eb' | 'cinzel' | 'raleway' | 'inter' | 'mono' | 'baloo'
Object.keys(ACCENT_COLORS);  // 'Altın' | 'Gümüş' | 'Mor' | 'Mavi' | 'Yeşil' | 'Kırmızı'
```

## Lisans

MIT
