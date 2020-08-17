# Tools

Not all tools are equally useful, but you might be interested in seeing how they work with the various file formats if your desire is to learn more about them.

## CuriousKitten's GNDEdit

Highly recommended to view all the intricate map details; no tool I've found gives a more comprehensive overview. Unfortunately it doesn't explain what any of the information means, so it's not as useful as it could be and can be a bit overwhelming at first.

[Link @ dotalux.org (mirror)](https://dotalux.com/ro/GNDedit/)

## Tokei's GRF Editor

The most modern file viewer I could find, which even supports map rendering! Unfortunately, it doesn't seem to show the data for the complex structures that it parses, such as the GND's cube geometry itself.

[Link @ rathena.org](https://rathena.org/board/files/file/2766-grf-editor/)

## Tokei's ACT Editor

Uses GRF Editor, but provides a better interface for viewing animations. Also supports anchor points, despite not really explaining their use.

[Link @ rathena.org](https://rathena.org/board/files/file/3304-act-editor/)

## Borf's BrowEdit

It's a map editor, which allows you to edit maps. I don't think it's very stable and the UI is truly horrendous, but nevertheless there might be some situations where playing around with it can help you to understand the map structures better.

Since there's so many different versions out there and most of them seem to randomly crash, I can't give a specific link that is guaranteed to work. You can try compiling the source obtained [on GitHub](https://github.com/Borf/browedit) and hope for the best.

If someone can point me to a stable version of this I'll happily add the link. However, from what I've seen, such a version does not exist.

## Vincent Thibault's RO Browser

Not a tool by itself, this is an app designed to mimic the original Ragnarok client's behaviour, which you can run in your browser. In order to achieve this incredible feat, it had to deal with virtually ALL of the proprietary file types, which is probably why it was taken down by Gravity in the first place.

While I've no idea what happened there, the source code is once again available on GitHub and since it's written in JavaScript doesn't require any compilation. You can simply open the files and read what's inside of them, or change things around to see how it affects the game world.

So in a way, it is the ultimative tool to aid in understanding the file formats, or it would be if the source was more readily understandable and the things it does much better documented than is unfortunately the case.

[Link @ GitHub](https://github.com/vthibault/roBrowser)

## curiosity's RagnarokJS Browser client

Very similar to RoBrowser, though less complete but slightly easier to understand. It's not really documented either, unfortunately, so you'll have to do a lot of digging either way.

[Link @ GitHub](https://github.com/GodLesZ/rangarok-js)

[Backup fork @ GitHub](https://github.com/SacredDuckwhale/rangarok-js)