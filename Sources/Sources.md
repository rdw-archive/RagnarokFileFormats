# Sources

## Documentation (sort of)

Obviously, there's no proper documentation to be found anywhere, which is why this repository exists.

But some people have written about at least some of their findings:

* [Rick Sarvas' essay on early-day map editing problems](http://web.archive.org/web/20071015031008/http://www.neatocool.com/Projects/Ragnarok/Map_Editors_Text/): Screenshots weren't archived and I don't have them either, but it's the only "intuitive" explanation of the high-level concepts I've found.
* [RagnarokJS thread @ rathena](https://rathena.org/board/topic/104827-wip-native-ragnarok-client/): Some research was shared here.
* [RoBrowser thread @ rathena](https://rathena.org/board/topic/53323-robrowser-ragnar%C3%B6k-online-in-browser/): Some research was shared here.
* [Shinryo's native client thread @ rathena](https://rathena.org/board/topic/57955-custom-ragnarok-online-client/): Some research was shared here.
* [Fimbulwinter client thread @ rathena](https://rathena.org/board/topic/74415-fimbulwinter-client/): Some research was shared here.

## Specifications

Most of them are incomplete, outdated, or plain wrong, but they were a good starting point to kick off this project.

* [Ximosoft's Ragnarok Online Laboratory](https://web.archive.org/web/20100224134600/http://rolaboratory.ximosoft.com/): Seems outdated and possibly wrong/incomplete in many parts.
* [Gratia Huang's website](http://mist.in/gratia/ro/): ACT/SPR details, requires Google Translate and manual conversion to Unicode unless you speak Chinese. Surprisingly detailed, if a bit contradictory at times.
* [FlavioJS's notes](https://github.com/flaviojs/eathena-devel-FlavioJS/tree/master/client/file_formats), which were very helpful to understand the various differences introduced in later versions of many file types. I think there are some errors, but it's a start.
* [OpenRagnarok file formats specification](https://web.archive.org/web/20090911093940/http://www.open-ragnarok.org/cms/file-formats): Nothing new here, but should be fairly accurate for the original file formats.
* [VCS's notes on field files](http://openkore.com/index.php/Field_file_format): Some more details on GAT terrain types, although sorely lacking in detail.

## Third-party client projects

All of these are unfinished and poorly (or not at all) documented, but some of the technical details can still be gleaned if you spend a lot of time and reference them against each other.

* [Vincent Thibault's RoBrowser](https://github.com/vthibault/roBrowser): ALL the file types in one place, but poorly documented. Much of the juiciest parts are unfortunately utterly incomprehensible and very low-level.
* [curiosity's ragnarok-js](https://github.com/GodLesZ/rangarok-js): Not sure if this is the right repository, but I haven't found anything that looked "official". It uses a rendering framework, so even if there's no documentation some knowledge can be extracted from using the framework's documentation, at least.
* [Doddler's RagnarokRebuild](https://github.com/Doddler/RagnarokRebuild): Most useful is probably the [Twitter feed](https://twitter.com/RoDoddler/), since the project itself doesn't have documentation and also uses Unity, which makes it less suitable for learning.
* [Guilherme Hernandez' UnityRO](https://github.com/guilhermelhr/unityro): No documentation and also uses Unity, but another reference implementation can't hurt.
* [Temtaime's Aesir](https://github.com/Temtaime/aesir): They seem to have done a lot of research, but didn't document anything. I really wish they'd have shared their findings and used a less obscure language... Oh, well.
* [bbodi's Rustarok](https://github.com/bbodi/rustarok): Not really a Ragnarok client, but uses the same file formats so it might be useful?
* [OpenRagnarok](https://github.com/open-ragnarok/roint): Abandoned and not very helpful, possibly wrong? But it's another attempt that can be used for comparison.
* [greenboxal's Fimbulwinter Client](https://github.com/greenboxal/fimbulclient): Also unfinished, undocumented, and abandoned. One more to add to the list, I guess?