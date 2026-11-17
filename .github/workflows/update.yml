"""Build arabic.m3u: Arabic-language channels from iptv-org, grouped by country, sorted A-Z."""
import re
import urllib.request

LANG_URL = "https://iptv-org.github.io/iptv/languages/ara.m3u"
COUNTRY_URL = "https://iptv-org.github.io/iptv/index.country.m3u"
OUT = "arabic.m3u"


def fetch(url):
    req = urllib.request.Request(url, headers={"User-Agent": "Mozilla/5.0"})
    with urllib.request.urlopen(req, timeout=120) as r:
        return r.read().decode("utf-8", "replace")


def parse(text):
    """Yield (extinf, extra_lines, url) for each entry in an M3U."""
    extinf, extra = None, []
    for line in text.splitlines():
        line = line.strip()
        if not line or line.startswith("#EXTM3U"):
            continue
        if line.startswith("#EXTINF"):
            extinf, extra = line, []
        elif line.startswith("#"):
            if extinf:
                extra.append(line)  # e.g. #EXTVLCOPT user-agent / referrer
        elif extinf:
            yield extinf, extra, line
            extinf, extra = None, []


def attr(extinf, name):
    m = re.search(name + r'="([^"]*)"', extinf)
    return m.group(1) if m else ""


def title(extinf):
    return extinf.rsplit(",", 1)[-1].strip()


def set_group(extinf, group):
    if 'group-title="' in extinf:
        return re.sub(r'group-title="[^"]*"', f'group-title="{group}"', extinf)
    head, name = extinf.rsplit(",", 1)
    return f'{head} group-title="{group}",{name}'


def main():
    # stream URL -> set of countries it is listed under
    countries = {}
    for extinf, _, url in parse(fetch(COUNTRY_URL)):
        for g in attr(extinf, "group-title").split(";"):
            if g.strip():
                countries.setdefault(url, set()).add(g.strip())

    entries, seen = [], set()
    for extinf, extra, url in parse(fetch(LANG_URL)):
        for country in countries.get(url, {"Undefined"}):
            if (country, url) in seen:
                continue
            seen.add((country, url))
            entries.append((country, title(extinf), set_group(extinf, country), extra, url))

    # Countries A-Z ("Undefined" last), then channels A-Z
    entries.sort(key=lambda e: (e[0] == "Undefined", e[0].lower(), e[1].lower()))

    out = ["#EXTM3U"]
    for _, _, extinf, extra, url in entries:
        out += [extinf, *extra, url]
    with open(OUT, "w", encoding="utf-8") as f:
        f.write("\n".join(out) + "\n")
    print(f"Wrote {len(entries)} entries in {len({e[0] for e in entries})} countries")


if __name__ == "__main__":
    main()
