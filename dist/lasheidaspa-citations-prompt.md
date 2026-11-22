Build a Google Sheet that lists the online citations and branded profiles for this local business. I'll embed the sheet publicly on the business's ID page, so accuracy matters more than length.

BUSINESS (use exactly as written)
Name: La Sheida Spa
Address: 1601 Village Market Blvd SE, Suite 100, Leesburg, VA 20175
Phone: 571-218-0986
Website: https://lasheidaspa.com/
Category: Day Spa (eyebrow threading, facials, waxing)
Google Business Profile: https://maps.google.com/maps?cid=15452416542071472731

KNOWN ASSETS (include these first)
- Website: https://lasheidaspa.com/
- Google Business Profile: https://maps.google.com/maps?cid=15452416542071472731
- Facebook: https://www.facebook.com/LASHEIDASPA/
- Instagram: https://www.instagram.com/lasheidaspa/
- TikTok: https://www.tiktok.com/@lasheida
- ID page: https://bigstack.s3.amazonaws.com/lasheidaspa/id.html
- Google Cloud page: https://storage.googleapis.com/g-bigstacks/La-Sheida-Spa.html
- BatchGeo map: https://batchgeo.com/map/c5e3f9441a7dcd23182954e4c1d5a2fc

THEN SEARCH FOR EXISTING LISTINGS on: Yelp, Apple Maps, Bing Places, Yellow Pages, BBB, Foursquare, MapQuest, Nextdoor, Manta, the local Chamber of Commerce, Booksy/Vagaro/StyleSeat/Fresha (booking sites), Salon Lofts (the business is inside Salon Lofts at the Village at Leesburg), and any local Leesburg or Loudoun County directories.

RULES
- Only include a listing if you found the actual page and it is for this business. Never guess or construct a URL.
- If a listing exists but shows an old address or phone number, include it and mark NAP Match = "No". The business recently moved, so old-address listings are likely.
- If you're unsure a listing is this business, put it on a second tab named "To verify" instead of the main tab.

COLUMNS (main tab, named "Citations")
A: Asset Name (e.g., "Yelp")
B: Type (Website, Google, Social, Directory, Review Site, Booking, Map, Stack Page)
C: URL
D: NAP Match (Yes / No / Partial)
E: Notes (what's wrong if NAP doesn't match)
F: Keyword (cycle through: Day Spa Leesburg, Eyebrow Threading, Eyebrow Threading Leesburg, Facial Spa Leesburg, Esthetician Leesburg, Waxing Leesburg, Eyebrow Shaping)
G: Linked Keyword, as a formula: =HYPERLINK(C2, F2) filled down

FORMAT
- Header row bold and frozen; sort by Type, with Website and Google first.
- Put a title row above the header: "La Sheida Spa – Citations & Branded Assets".
