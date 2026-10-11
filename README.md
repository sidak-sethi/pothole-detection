Roadwatch — Road Surface Monitoring
Roadwatch is a browser-based road defect monitoring dashboard designed around a Greater Chennai Corporation workflow. It brings defect locations, severity, model confidence, review status, and maintenance actions into one interface to help teams review and prioritise reported road-surface issues.
Features
- Interactive map: Leaflet map with CARTO basemap tiles and clickable, severity-coloured defect markers.
- Defect overview: Summary cards for items needing attention, items in review, scheduled repairs, repaired records, and total records in the selected time range.
- Severity breakdown: Counts and relative bars for minor, moderate, severe, and critical defects.
- Detection trend: Defect counts over 24-hour, 7-day, and 30-day time ranges.
- Search and filters: Search by road, area, or defect ID; filter by attention status, critical severity, scheduled work, or repaired status.
- Sortable records: Sort the detection table by severity, confidence, or detection time.
- Defect detail drawer: Inspect a record and use available workflow actions, such as scheduling, marking repaired, dismissing, or restoring a record.
- Responsive interface: Layout adapts to smaller screens, with a collapsible navigation sidebar.
- Theme preference: Switch between light and dark themes; the theme preference is stored locally in the browser.
Technology Stack
- HTML5 and CSS3
- Vanilla JavaScript
- Leaflet for interactive maps
- CARTO Basemaps for map tiles
- OpenStreetMap geographic data and attribution
- Google Fonts: IBM Plex Sans and Space Grotesk
The current front end is a static HTML application. It does not require a build step or package installation.
Project Structure
Roadwatch/
├── index.html     # Dashboard markup, styles, JavaScript, and initial records
└── README.md      # Project documentation
If you have additional project artefacts, such as the conference paper, poster, or invention disclosure, keep them in a separate docs/ folder or the project submission directory.
Run Locally
Option 1: VS Code Live Server
1. Place index.html and README.md in the same project folder.
2. Open the folder in Visual Studio Code.
3. Install the Live Server extension if needed.
4. Right-click index.html and choose Open with Live Server.
5. Open the local URL displayed by VS Code, commonly http://127.0.0.1:5500/index.html.
Option 2: Python local server
Open a terminal in the project folder and run:
python -m http.server 8000
Then open http://localhost:8000 and select index.html.
An internet connection is required to load Leaflet, fonts, and the basemap tiles from their CDNs/providers.
Map Configuration
The map is initialised in the JavaScript function initMap(). It uses Leaflet and a CARTO Voyager raster tile layer.
1. Create or access a CARTO account and obtain a basemap key through the provider's official setup process.
2. Locate the CARTO_API_KEY constant in index.html.
3. Configure it with the key issued for your application.
4. Apply provider-supported usage restrictions, including allowed website domains where available.
5. Reload the site and check that map tiles load and that the map attribution remains visible.
Key handling: A key embedded in a client-side HTML file is visible to anyone who can load the page. Do not place private server credentials in this file. Restrict browser keys to the intended domains and rotate any key that has been exposed unintentionally. Review the provider's current plan, rate limits, and terms before deployment.
Map resources:
- Leaflet Quick Start
- CARTO Basemaps
- OpenStreetMap tile usage policy
Defect Record Fields
Records are currently initialised in the JavaScript data array. Each record uses the following fields:
Field	Description
id	Unique defect identifier, such as PH-2098
street	Road name
area	Locality or area
lat, lng	Geographic latitude and longitude in decimal degrees
severity	One of minor, moderate, severe, or critical
confidence	Model confidence value represented from 0 to 1
detected	JavaScript Date representing detection time
status	Workflow state: new, review, scheduled, repaired, or dismissed
crew	Assigned maintenance crew, if applicable
depth, width	Defect measurements used in the detail view


Ensure that record IDs are unique, coordinates are valid, confidence values are between 0 and 1, and categorical values use the supported status and severity names. Use GPS coordinates from the capture device or a verified source for operational records.
Current Application Behaviour
- The front end builds its initial records from the JavaScript data array. It does not currently fetch defect records from a backend API.
- Search, filters, sorting, markers, summary cards, and chart-like summaries operate on the browser's in-memory records.
- Workflow actions update the current page state. A reload restores the records initialised by the source code because persistence for defect updates is not connected.
- The selected theme is saved through browser localStorage.
- The displayed confidence is a record field. It should only be described as a validated model score after its source model and evaluation have been documented.
Suggested Integration Architecture
A production deployment can connect the interface to a detection and maintenance pipeline:
Road image / video + GPS
          |
          v
Road-defect detection model
          |
          v
Detection API  --->  Database
          |              |
          v              v
Validated defect records + status history
          |
          v
Roadwatch dashboard and Leaflet map
          |
          v
Review / assign crew / repair / verify closure
Recommended API responsibilities:
- GET /api/defects: return defect records, including ID, coordinates, severity, confidence, time, and status.
- PATCH /api/defects/{id}: persist status and assignment changes after authorisation.
- GET /api/health: report service availability.
The API routes above are recommendations, not routes implemented by the current static page. Validate incoming coordinates and values on the server, record status changes with timestamps and user IDs, and protect write operations with authentication and authorisation.
Data Science and Validation Roadmap
The dashboard can be extended into a data-science workflow by adding:
1. Detection evaluation: Measure precision, recall, F1-score, and mean average precision on a held-out, labelled dataset. Report class-level results and confidence thresholds.
2. Location quality: Measure GPS error and reject missing or implausible coordinates before mapping.
3. Duplicate grouping: Cluster nearby detections on the same road within a time window; retain the original reports and link them to a common defect case.
4. Hotspot analysis: Aggregate verified defects by road segment or geographic grid, normalise rates by traffic exposure where reliable data exists, and identify recurring locations.
5. Maintenance prioritisation: Develop and evaluate a transparent risk score using severity, confidence, recurrence, road class, and exposure. Do not treat an unvalidated score as a safety guarantee.
6. Closure verification: Compare later images or inspections with the original defect and retain evidence for repaired or dismissed outcomes.
7. Operational monitoring: Track API availability, data freshness, inference latency, invalid-coordinate rate, duplicate rate, and repair turnaround time.
Do not publish performance figures until they have been measured using a documented dataset, evaluation split, and reproducible test procedure.
Testing Checklist
- [ ] Basemap tiles load, and attribution is visible.
- [ ] Map zoom and pan controls work.
- [ ] All defect markers appear at their stored coordinates and in the expected geographic area.
- [ ] Marker colour and outline match severity and repaired status.
- [ ] Clicking a marker opens the relevant defect details.
- [ ] Search returns expected IDs, road names, and areas.
- [ ] Each status/severity filter shows the expected records.
- [ ] Time-range controls update all relevant summaries consistently.
- [ ] Sorting by severity, confidence, and time works in both directions.
- [ ] Status actions update the interface as expected.
- [ ] Theme switching and responsive layout work.
- [ ] Missing map library, unavailable tile service, and invalid API configuration are handled clearly.
- [ ] Data updates survive a reload after a backend persistence layer is implemented.
Record expected outcome, actual outcome, date, browser/device, and evidence for every executed test.
Limitations and Responsible Use
- The current application is a browser-based monitoring and workflow interface; an automated image-detection service and a persistent backend are not connected by the static page itself.
- Map accuracy depends on the latitude and longitude attached to each record. Confirm each location before using it to dispatch maintenance teams.
- Public or third-party map tiles can have usage restrictions, availability limits, or rate limits. Do not assume unlimited service.
- A model confidence value is not a probability of harm unless calibrated and validated for that interpretation.
- Prioritisation should support, not replace, qualified municipal inspection and safety decisions.
- Avoid publishing personal data, vehicle registration plates, faces, or sensitive location information unless lawful and necessary.
Attribution
- Map library: Leaflet
- Map tiles: CARTO Basemaps
- Geographic data attribution: © OpenStreetMap contributors
- Fonts: IBM Plex Sans and Space Grotesk
Check each provider's current licence and terms before redistributing or deploying the application.
Licence
Open Source (MIT)
Add the project's intended source-code licence before public distribution. Unless a licence is included, do not assume the project is available for unrestricted reuse.
Maintainers
Add project members, department, institution, faculty guide, and a contact email here before publishing the repository.
