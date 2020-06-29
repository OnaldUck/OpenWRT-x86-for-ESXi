
### git Ordner aktualisieren

```
cd openwrt
git pull
./scripts/feeds update -a
./scripts/feeds install -a
```

bei Problemen mit `feeds.conf.default` - einfach die Datei löschen
