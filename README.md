
Headless Chrome kullanarak **alıntı kartları**, **Spotify kartları** ve **rank/seviye kartları** PNG buffer olarak üretir. Discord botları için yapıldı, her Node projesinde çalışır.

---

## Önizleme

### Alıntı Kartı — Sinema Stili
> ![Sinema](https://github.com/utw0/luhux-card/blob/main/preview-cinema.png)

### Alıntı Kartı — Poster Stili
> ![Poster](https://github.com/utw0/luhux-card/blob/main/preview-poster.png)

### Alıntı Kartı — Minimal Stili
> ![Minimal](https://github.com/utw0/luhux-card/blob/main/preview-minimal.png)

### Spotify Kartı
> ![Spotify](https://github.com/utw0/luhux-card/blob/main/preview-spotify.png)

### Rank Kartı
> ![Rank](https://github.com/utw0/luhux-card/blob/main/preview-rank.png)

---

## Kurulum

```bash
npm install luhux-card
```

`puppeteer-core` kullanır — Chromium **indirmez**, bilgisayarındaki kurulu Chrome'u otomatik bulur. Chrome kurulu değilse `PUPPETEER_EXECUTABLE_PATH` ortam değişkeniyle yolu belirtebilirsin.

---

## Alıntı Kartı

### Hızlı başlangıç

```ts
import { generateQuoteCard } from 'luhux-card';
import fs from 'fs';

const buffer = await generateQuoteCard({
    avatarUrl: 'https://cdn.discordapp.com/avatars/.../avatar.png',
    displayName: 'luhux1337',
    username: 'luhux',
    text: 'selamun aleyküm :D',
    style: 'cinema',       // 'cinema' | 'poster' | 'minimal'
    accent: '#10b981',
    bg: '#0a0a0f',
    fontKey: 'inter',
    watermark: 'my-bot.dev',
});

fs.writeFileSync('quote.png', buffer);
```

### Tekrarlı render (Discord butonları)

Kullanıcı butonlarla stil/renk değiştiriyorsa `QuoteCardRenderer` kullan — browser'ı açık tutar, her render çok daha hızlı olur.

```ts
import { QuoteCardRenderer } from 'luhux-card';

const renderer = new QuoteCardRenderer();

// Her buton tıklamasında:
const buffer = await renderer.render(state);

// Collector bittiğinde:
await renderer.close();
```

### Discord.js örneği

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

await renderer.close();
```

### Seçenekler (`QuoteCardOptions`)

| Seçenek | Tip | Varsayılan | Açıklama |
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
| `fontSize` | `number` | `22` | temel boyut; sinema modunda metin uzunluğuna göre otomatik ölçeklenir |
| `dim` | `number` (10–90) | `55` | fotoğrafın karartılma miktarı |
| `watermark` | `string` | `'luhux-card'` | sinema stilinde sol alt köşedeki yazı |
| `width` / `height` | `number` | `920` / `460` | tuval boyutu |
| `scale` | `number` | `2` | ekran görüntüsü ölçek çarpanı (netlik / hız dengesi) |

### Hazır presetler

```ts
import { FONTS, ACCENT_COLORS, BG_COLORS, TEXT_COLORS } from 'luhux-card';

Object.keys(FONTS);
// 'cormorant' | 'playfair' | 'eb' | 'cinzel' | 'raleway' | 'inter' | 'mono' | 'baloo'

Object.keys(ACCENT_COLORS);
// 'Altın' | 'Gümüş' | 'Mor' | 'Mavi' | 'Yeşil' | 'Kırmızı'
```

---

## Spotify Kartı

Spotify "şu an çalıyor" kartı — albüm kapağı, blur arka plan, progress bar ve öneri listesiyle.

### Hızlı başlangıç

```ts
import { generateSpotifyCard } from 'luhux-card';
import fs from 'fs';

const buffer = await generateSpotifyCard({
    title: 'Şarkı Adı',
    artist: 'Sanatçı',
    album: 'Albüm Adı',
    cover: 'https://i.scdn.co/image/...',
    username: 'luhux',
    startTime: Date.now() - 60_000,
    endTime: Date.now() + 180_000,
    recommendations: [
        { name: 'Başka Şarkı', artists: 'Başka Sanatçı', url: 'https://open.spotify.com/track/...' },
    ],
});

fs.writeFileSync('spotify.png', buffer);
```

### Discord.js örneği (presence üzerinden)

```ts
import { generateSpotifyCard } from 'luhux-card';
import { AttachmentBuilder, ActivityType } from 'discord.js';

const status = member.presence?.activities?.find(
    a => a.name === 'Spotify' && a.type === ActivityType.Listening
);

if (status) {
    const albumArt = `https://i.scdn.co/image/${status.assets?.largeImage?.slice(8)}`;

    const buf = await generateSpotifyCard({
        title: status.details ?? '',
        artist: status.state ?? '',
        album: status.assets?.largeText ?? '',
        cover: albumArt,
        username: member.displayName,
        startTime: status.timestamps?.start ?? Date.now(),
        endTime: status.timestamps?.end ?? Date.now(),
    });

    await interaction.reply({
        files: [new AttachmentBuilder(buf, { name: 'spotify.png' })],
    });
}
```

### Seçenekler (`GenerateSpotifyCardOptions`)

| Seçenek | Tip | Varsayılan | Açıklama |
|---|---|---|---|
| `title` | `string` | — | zorunlu, şarkı adı |
| `artist` | `string` | — | zorunlu, sanatçı adı |
| `album` | `string` | `''` | albüm adı |
| `cover` | `string \| Buffer` | — | albüm kapağı URL, base64 data URL veya dosya yolu |
| `username` | `string` | `'Kullanıcı'` | dinleyen kişinin adı |
| `startTime` | `number \| Date` | — | şarkının başladığı zaman (timestamp) |
| `endTime` | `number \| Date` | — | şarkının biteceği zaman (timestamp) |
| `progressMs` | `number` | — | geçen süre ms (startTime/endTime yerine kullanılabilir) |
| `durationMs` | `number` | — | toplam süre ms (startTime/endTime yerine kullanılabilir) |
| `recommendations` | `SpotifyCardRecommendation[]` | `[]` | benzer şarkı önerileri (en fazla 3) |
| `theme` | `SpotifyCardTheme` | — | renk özelleştirme |
| `theme.accent` | `string` | `'#1DB954'` | Spotify yeşili (progress bar rengi) |
| `theme.accentLight` | `string` | `'#20e565'` | progress bar gradient sonu |
| `theme.background` | `string` | `'#0f0f0f'` | kart arka plan rengi |
| `labels` | `SpotifyCardLabels` | — | metin özelleştirme |
| `labels.nowPlaying` | `string` | `'Şu anda çalıyor'` | badge yazısı |
| `labels.listener` | `string` | `'Dinleyen'` | dinleyen etiketi |
| `labels.suggestions` | `string` | `'Benzer Şarkı Önerileri'` | öneri başlığı |
| `width` | `number` | `780` | kart genişliği |
| `scale` | `number` | `2` | ekran görüntüsü ölçek çarpanı |
| `outputPath` | `string` | — | belirtilirse dosyaya da kaydeder |

---

## Rank Kartı

Kullanıcı seviye/XP kartı — avatar, sunucu sırası, istatistikler, XP progress bar ve rozetlerle.

### Hızlı başlangıç

```ts
import { generateRankCard } from 'luhux-card';
import fs from 'fs';

const buffer = await generateRankCard({
    displayName: 'luhux1337',
    username: 'luhux',
    rank: 4,
    level: 38,
    xp: 72_450,
    levelStartXp: 50_000,
    nextLevelXp: 100_000,
    messageCount: 1284,
    voiceHours: 62,
    streak: 14,
    accent: '#7c3aed',
    badges: [
        { label: 'Aktif Üye',     color: 'purple', emoji: '⚡' },
        { label: 'Top 10',        color: 'gold',   emoji: '🏆' },
        { label: 'Sohbet Ustası', color: 'blue',   emoji: '💬' },
        { label: 'Ses Dostu',     color: 'green',  emoji: '🎵' },
    ],
});

fs.writeFileSync('rank.png', buffer);
```

### Discord.js örneği

```ts
import { generateRankCard } from 'luhux-card';
import { AttachmentBuilder } from 'discord.js';

const buf = await generateRankCard({
    avatarUrl: member.displayAvatarURL({ size: 512, extension: 'png' }),
    displayName: member.displayName,
    username: member.user.username,
    rank: 4,           
    level: 38,
    xp: 72_450,
    levelStartXp: 50_000,
    nextLevelXp: 100_000,
    messageCount: 1284,
    voiceHours: 62,
    streak: 14,
    accent: '#7c3aed',
});

await interaction.reply({
    files: [new AttachmentBuilder(buf, { name: 'rank.png' })],
});
```

### Seçenekler (`RankCardOptions`)

| Seçenek | Tip | Varsayılan | Açıklama |
|---|---|---|---|
| `displayName` | `string` | — | zorunlu, görünen ad |
| `level` | `number` | — | zorunlu, mevcut seviye |
| `xp` | `number` | — | zorunlu, mevcut toplam XP |
| `nextLevelXp` | `number` | — | zorunlu, sonraki seviye için gereken XP |
| `avatarUrl` | `string` | — | avatar URL (yoksa baş harf gösterilir) |
| `username` | `string` | — | `@handle` olarak gösterilir |
| `rank` | `number` | — | sunucu sırası |
| `levelStartXp` | `number` | `0` | bu seviyenin başlangıç XP'si |
| `messageCount` | `number` | — | toplam mesaj sayısı |
| `voiceHours` | `number` | — | toplam ses saati |
| `streak` | `number` | — | günlük aktiflik serisi |
| `badges` | `RankCardBadge[]` | — | rozetler (aşağıya bak) |
| `accent` | hex renk | `'#7c3aed'` | kart accent rengi |
| `width` | `number` | `780` | kart genişliği |
| `scale` | `number` | `2` | ekran görüntüsü ölçek çarpanı |
| `outputPath` | `string` | — | belirtilirse dosyaya da kaydeder |

### Rozetler (`RankCardBadge`)

| Alan | Tip | Açıklama |
|---|---|---|
| `label` | `string` | rozet yazısı |
| `color` | `'purple' \| 'gold' \| 'blue' \| 'green' \| 'red'` | rozet rengi |
| `emoji` | `string` | isteğe bağlı emoji |

---

## Lisans

MIT © Utku 'luhux' KÖSEM
