Build a Google Sheet that lists the online citations and branded profiles for this local business. I'll embed the sheet publicly on the business's ID page, so accuracy matters more than length.

BUSINESS (use exactly as written)
Name: {{name}}
Address: {{address}}
Phone: {{phone}}
Website: {{website}}
Category: {{category}}
Google Business Profile: {{gbp}}

KNOWN ASSETS (include these first)
{{knownAssets}}

THEN SEARCH FOR EXISTING LISTINGS on: Yelp, Apple Maps, Bing Places, Yellow Pages, BBB, Foursquare, MapQuest, Nextdoor, Manta, the local Chamber of Commerce, {{extraSources}}and any local {{city}} or {{region}} directories.

RULES
- Only include a listing if you found the actual page and it is for this business. Never guess or construct a URL.
- If a listing exists but shows an old address or phone number, include it and mark NAP Match = "No".{{napNote}}
- If you're unsure a listing is this business, put it on a second tab named "To verify" instead of the main tab.

COLUMNS (main tab, named "Citations")
A: Asset Name (e.g., "Yelp")
B: Type (Website, Google, Social, Directory, Review Site, Booking, Map, Stack Page)
C: URL
D: NAP Match (Yes / No / Partial)
E: Notes (what's wrong if NAP doesn't match)
F: Keyword (cycle through: {{keywords}})
G: Linked Keyword, as a formula: =HYPERLINK(C2, F2) filled down

FORMAT
- Header row bold and frozen; sort by Type, with Website and Google first.
- Put a title row above the header: "{{name}} – Citations & Branded Assets".
