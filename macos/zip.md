---
description: MacOS
---

# Command Line Reference

Bij het aanmaken van een zip-file op een Mac wordt een \_\_MACOSX directory toegevoegd. Niemand weet waarom. Om deze folder uit het zip-archief te verwijderen kun je het volgende commando gebruiken.

```
zip -d your-archive.zip "__MACOSX*"
```

> Met onderstaand commando kan de inhoud van een archief worden bekeken, zonder daadwerkelijk uit te pakken.
>
> ```
> unzip -l your-archive.zip
> ```
