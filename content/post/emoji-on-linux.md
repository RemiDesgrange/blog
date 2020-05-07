---
title: "Emoji on Linux"
date: 2020-05-07T16:33:32+02:00
draft: false
tags: ["Linux", "Emoji", "System"]
category: ["Linux", "Emoji", "System"]
---

I tried several stuff to configure emoji properly on Linux. I really suffer to get it working

Here is my desktop setup :

* Distro: Arch Linux
* WM: Wayland
* DE: Gnome
* Terminal: [Alacritty](https://github.com/alacritty/alacritty)

If you don't know Alacritty, you should check it out. They still have a [weird bug](https://github.com/alacritty/alacritty/issues/1864) with emoji but it's a great terminal emulator.

I installed the noto emoji, but it emoji one also works. 

On arch:

```
pacman -S noto-fonts-emoji
```

On ubuntu

```
apt install fonts-noto
```

Then in `~/.config/fontconfig/fonts.conf` add this config

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
  <alias>
    <family>serif</family>
    <prefer>
      <family>Noto Color Emoji</family>
    </prefer>
  </alias>
  <alias>
    <family>sans-serif</family>
    <prefer>
      <family>Noto Color Emoji</family>
    </prefer>
  </alias>
  <alias>
    <family>monospace</family>
    <prefer>
      <family>Noto Color Emoji</family>
    </prefer>
  </alias>
</fontconfig>
```


You'll need to regenerate fontconfig cache with `fc-cache -f -v`.

## It's not working

Fontconfig is a bit cumbersome to configure. I had trouble with alacrity but it worked in gnome terminal or Firefox. You can troubleshoot with this command:

```bash 
fc-match -s <YourFont>
# example for me
fc-match -s OperatorMono
```

If `Noto` is at bottom of the list, then you probably have some fontconfig issue.



## Can you see it ? 

♑  ↗  💁  💼  🌔  🔎  📙  🛰  ♓  👬  🍣  ⏳  ㊙  📏  🐴  🙂  🎺  🎾  🎣  📼  👃  🏣  🐮  🏭  💧  🌉  😌  😬  ↙  ↩  🏴  🚍  🏞  🚴  🍁  🐣  🌪  📷  ♻  😾  🌷  🐬  🐷  💻  ⬅  🕧  🔼  🔚  🛌  🛥  🗡  🕟  🖊  ♊  🕙  ⛺  🇰🇷  👞  🍍  ☔  🍽  🏝  🛣  🐚  🚞  ☃  🎪  🛐  🍰  🍼  🏫  🐯  🔘  ⬆  🛬  ⚗  🚺  🈴  😭  🕰  ⤵  🌳  ✡  🌥  ⁉  🍗  👵  🏄  🚂  👎  🙉  ⏱  😲  📵  🚝  🎶  💞  ☝  🃏  🐩  💾  ❔  🚱  ➰  🙏  🐓  🈯  🈷  🈶  👋  ☁  ⛩  ✍  🏧  🦃  🕗  🐗  💌  🐫  😗  🏆  🏵  👉  📬  🏓  🎿  😨  👠  🍹  🦀  🐈  👂  🌹  😢  👨  🎆  👶  🚈  🔅  📆  🎌  🐏  🖼  💦  💉  🐎  💢  🏊  😕  🎅  🐀  😵  🐁  0⃣  🛋  🍾  📫  🎮  📸  ↔  🚩  🎐  🏉  🗻  🎨  🔽  🇩🇪  🖖  🈚  🌡  👄  🌾  👆  🚦  🕋  🎻  🕍  🍯  🖖  ⛳  🌺  🍄  🐞  🐸  💈  🍙  🚫  🌞  ⚡  😜  😉  💐  ♣  🎏  🐶  🤘  ❣  👒  😞  🈸  🐼  🏇  🙄  🎰  ⛎  🌖  🖍  📤  🍿  👗  👳  🔤  ⏮  Ⓜ  🏏  🏺  🖇  🛏  🌸  ♿  ⚒  ⌚  🏟  🐭  🚙  😳  🌦  🏪  🔋  🍤  🔄  ☯  🍉  📑  ⚙  ♌  🐃  💷  🖨  💋  ▪  ⚛  🐺  🏜  🔲  🎥  🚭  🔑  ✌  ⏬  🍳  🔰  ⛈  ☀  🍭  🐐  ✈  🍩  🏻  📜  💫  🍬  🕹  🕵  🈹  ✋  ❇  🎁  ☘  🍨  ⚽  👤  💵  🛳  🔀  ⬇  🐤  🏷  🙆  🍥  🔗  🆑  👌  😽  📌  ↘  🗂  🚯  💜  ♒  🎧  🔛  📍  💿  📟  🏤  🕉  🎳  🤐  ☂  #⃣  📰  ⚾  💶  🖋  🔓  🎚  🔫  👽  🍀  🎛  🗣  📔  👫  🐢  👇  🐹  🎎  😫  👚  🐪  🏎  🎲  🚚  🙅  😁  🎇  💓  🐙  🚁  😚  🌙  👛  💰  😋  👹  🕖  🕐  ⛅  🚕  👀  ⏭  ❎  🍊  🌤  📉  🍑  💒  🏥  🍡  🎼  👑  ⛰  🎩  🖲  👭  🍕  ☮  ⚔  🐊  🤓  📞  ⌛  🈲  📖  🌕  🐖  😎  🈂  🈵  👷  ☣  🏹  🍫  🕕  🤔  😥  🌚  ✊  😻  🚸  🚵  ➕  🚡  ♥  🍅  🌩  🍒  🔴  🎱  🕴  ⏫  ‼  🕌  🔺  🌧  🎄  🏔  🏮  🆕  🕸  🕤  🕠  🔏  🐻  🔝  🐳  👊  🚘  🌛  7⃣  🏋  🚃  😓  📻  🍠  📋  💽  ❗  🈳  🙊  📨  🎭  📒  🌑  🧀  ⏹  🚲  🔹  ✳  🚥  🔔  🎡  6⃣  👺  ⚱  🍇  😸  🈁  💚  🌠  😧  ⚠  😮  🐠  👍  🕶  ©  🕛  🔆  ♠  ◼  🙃  🕓  ⛵  🍴  👅  🚼  🏒  ✂  💑  🇪🇸  🚤  😘  😱  😼  🚣  📹  🌼  🎯  🙍  💣  🆔  🎓  *⃣  😹  ✔  🌭  🐂  😔  🔒  📯  🚔  🔱  🎒  🍷  🦂  😂  💱  ⏩  🌟  🐘  🏈  🤒  🚨  🌶  😍  🏦  🌃  3⃣  📊  👿  ↕  🐲  📪  🎙  🍎  🔷  ♨  🐑  🕝  🚒  🇮🇹  🏛  📥  🔶  💥  🏖  📐  ☪  ➿  🛄  🐜  🛍  👼  🐆  🚹  🍐  🚳  ❄  ⏲  ⚜  📈  🚏  📲  📠  😑  🗄  🐔  🏐  🚐  🐕  🕦  ⚖  🐅  🌫  🏀  😪  〽  🚉  🍸  🏙  🍶  📄  ▫  🐄  👦  🅰  🔈  🅱  🎸  1⃣  🍲  🎬  💮  😒  🚗  🌒  🏠  😄  🆙  📛  🎽  🕞  🔬  ❌  👈  ⚰  ⛓  ▶  ⏯  🅾  💘  🅿  💟  📧  🚰  🕊  🏬  🔌  🈺  🍏  🔟  👱  🏂  ⛏  🚀  💴  🔦  🌀  😶  🔂  👰  🏩  🍦  🚪  ⭐  ⚓  ♦  8⃣  🚷  ⛷  🗿  ⏏  🗃  ☺  🐦  👩  ❕  ◀  ⛲  🏳  🍢  🗝  ♍  🕳  🚛  🚋  🆖  🔳  👥  🛤  💺  🚟  🚓  🌋  🔣  🍓  🛫  🌨  🖱  🐉  🍔  🐿  🐵  🕥  🎫  🏘  🍖  4⃣  😙  ⛸  🌅  🍝  💝  🖐  💤  ❤  🆗  🐥  📮  👾  🍆  🚊  💳  🎤  🐇  9⃣  5⃣  🆓  ♏  🌓  🏗  🙈  👢  👕  🗳  🍞  🛀  🙋  👝  🌄  💂  🐰  ⛑  🛎  ♉  🙎  🔐  ✨  ➡  😐  ◽  🚮  😀  💩  📦  ♋  🖕  🔯  💆  🌍  📇  👖  🉐  🕎  🔍  🎍  👓  ⬛  📣  ☹  🎃  📗  👟  😆  💬  🏼  📎  💕  🏽  🔠  🏾  📭  🏿  😷  🗾  📕  😴  🇺🇸  🌮  🆘  💸  🏸  🕣  💃  ⛽  🎖  🛃  🔜  ⛔  🚖  🏑  😿  🐋  🖌  🌌  🇫🇷  ↖  🍚  ☄  😛  🍈  🍋  🏨  ⏪  🍘  🎟  🙇  ⛱  ☑  🐛  😝  🦁  ☦  ⤴  ✖  🌽  😈  🀄  🗼  🔥  🔧  🌘  💠  🚌  🍧  ⚫  🔸  🚠  🗜  😣  ➖  🚻  🔮  🔁  📀  🏌  💎  😊  👜  🍛  🐟  📡  🌱  📂  🙀  👧  😠  😅  🐝  🎗  💅  🐒  ↪  🐽  💨  🎷  🔖  🐧  🉑  🇯🇵  🌴  🛢  📁  🛂  🌁  🔕  ☠  ✝  ❓  🌿  🚅  🎵  🛩  🛅  🕷  📶  🔊  📴  🍜  👏  📝  😺  💇  🔪  📱  ◻  ⚪  🌈  🌲  🤖  🗒  ♈  🚎  🗯  🐌  🗓  🙌  🔨  📅  😰  🕘  🕔  🍵  📺  💄  🗽  🚇  👸  ⛴  💭  ™  💀  ⛹  🌂  👐  ♐  🚽  ➗  ㊗  👯  😩  🍮  👙  🤗  ⭕  👣  🍪  🎀  ⛪  🌇  🎠  🎹  🐡  😇  📢  🌵  👁  🔭  🎉  2⃣  〰  🍺  🐱  👲  🌯  🖥  🆎  🍂  💲  🎋  🦄  🍟  🚶  💊  📘  🐍  😖  🚢  😡  👮  🏢  🚧  💙  🔡  🏚  🆚  🗑  ®  📓  🆒  🔵  👔  🎞  🙁  🕯  🍻  🐾  👡  😟  🎂  ☕  🔻  👴  🎑  ☸  ℹ  🎊  🔩  ☢  🤕  📚  🌎  🔉  🍌  🏯  ◾  🇷🇺  🔢  🎈  🐨  🔇  ♎  🕑  🔙  🎦  🏅  🏁  🇨🇳  ⌨  🛠  📳  💔  🏍  ✒  ✴  😃  💏  🏃  ⛄  🕢  🚆  🕜  ✉  📽  🌐  🎢  🤑  🍃  🌗  👻  🌻  ⏸  🚾  🚬  📃  👘  🌰  🎴  💛  🏡  😦  💗  🌬  😏  🕚  🔃  ⏺  🌝  🗞  🏰  💹  💯  ✏  🌏  ☎  🍱  👪  ✅  🗺  💡  📩  🇬🇧  🚑  🛁  🕒  🚄  💪  😯  🌆  🕡  💍  📿  🌊  🚿  🔞  ⏰  🌜  🛡  😤  ⬜  💖  🚜  🏕  