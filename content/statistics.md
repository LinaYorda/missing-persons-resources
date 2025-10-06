---
title: "Global Coverage Map"
---

This interactive world map shows all countries where I have collected missing persons resources and information. Countries highlighted in **green** have available resources in my database, while countries in **gray** currently have no listed resources.

The map provides a visual overview of my database's geographical coverage, helping users quickly identify which countries have missing persons support services available.

<div id="coverage-map" style="height: 600px; width: 100%; border: 1px solid #ccc; border-radius: 8px; margin: 20px 0;"></div>

## Database Overview

- **Total Countries with Resources:** 19
- **Total Resources Listed:** 82 websites and databases
- **Geographical Coverage:** Europe, North America, Oceania, Asia, and Africa

### Featured Countries
My database includes comprehensive coverage for major countries like Germany (17 resources), United States (14 resources), and Canada (10 resources), as well as emerging coverage in countries across multiple continents.

*Click on any country to see available resources and get more information.*

<!-- Leaflet CSS -->
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />

<!-- Leaflet JavaScript -->
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

<script>
// Initialize the map
var map = L.map('coverage-map').setView([20, 0], 2);

// Add OpenStreetMap tiles
L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '© <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
}).addTo(map);

// Countries with resources (highlight in green) - using GeoJSON country names
var countriesWithResources = [
    "Australia", "Austria", "Brazil", "Canada", "France", "Germany", 
    "China", "Ireland", "Italy", "Netherlands", "New Zealand", "Poland", 
    "Portugal", "Romania", "South Africa", "Spain", "Switzerland", 
    "United Kingdom", "United States of America"
];

// Resource counts for tooltips - map from GeoJSON names to our display names
var resourceCounts = {
    "Australia": 8,
    "Austria": 2,
    "Brazil": 8,
    "Canada": 10,
    "France": 3,
    "Germany": 17,
    "China": 1, // Hong Kong
    "Ireland": 1,
    "Italy": 2,
    "Netherlands": 2,
    "New Zealand": 1,
    "Poland": 1,
    "Romania": 1,
    "South Africa": 1,
    "Spain": 2,
    "Switzerland": 7,
    "United Kingdom": 4,
    "United States of America": 14
};

// Display names for labels (our preferred names)
var displayNames = {
    "Australia": "Australia",
    "Austria": "Austria", 
    "Brazil": "Brazil",
    "Canada": "Canada",
    "France": "France",
    "Germany": "Germany",
    "China": "Hong Kong",
    "Ireland": "Ireland",
    "Italy": "Italy",
    "Netherlands": "Netherlands",
    "New Zealand": "New Zealand",
    "Poland": "Poland",
    "Romania": "Romania",
    "South Africa": "South Africa",
    "Spain": "Spain",
    "Switzerland": "Switzerland",
    "United Kingdom": "United Kingdom",
    "United States of America": "United States"
};

// Function to get country color
function getCountryColor(countryName) {
    return countriesWithResources.includes(countryName) ? '#90EE90' : '#cccccc';
}

// Function to get country fill opacity
function getCountryOpacity(countryName) {
    return countriesWithResources.includes(countryName) ? 0.7 : 0.3;
}

// Load world countries GeoJSON
fetch('https://raw.githubusercontent.com/johan/world.geo.json/master/countries.geo.json')
    .then(response => response.json())
    .then(data => {
        L.geoJSON(data, {
            style: function(feature) {
                return {
                    fillColor: getCountryColor(feature.properties.name),
                    weight: 1,
                    opacity: 1,
                    color: 'white',
                    fillOpacity: getCountryOpacity(feature.properties.name)
                };
            },
            onEachFeature: function(feature, layer) {
                var countryName = feature.properties.name;
                var hasResources = countriesWithResources.includes(countryName);
                var resourceCount = hasResources ? resourceCounts[countryName] || 0 : 0;
                var displayName = displayNames[countryName] || countryName;

                // Add permanent country labels for countries with resources
                if (hasResources) {
                    // Calculate centroid of the country for label placement
                    var centroid = getCentroid(feature.geometry.coordinates, feature.geometry.type);

                    if (centroid) {
                        L.marker(centroid, {
                            icon: L.divIcon({
                                className: 'country-label',
                                html: `<div style="background: rgba(255,255,255,0.8); padding: 2px 6px; border-radius: 3px; font-size: 12px; font-weight: bold; border: 1px solid #333;">${displayName}</div>`,
                                iconSize: null
                            })
                        }).addTo(map);
                    }
                }

                var popupContent = `
                    <strong>${displayName}</strong><br>
                    ${hasResources ? 
                        `Resources: ${resourceCount}<br><a href="/countries/${displayName.toLowerCase().replace(/\s+/g, '-')}/">View Resources</a>` : 
                        'No resources listed yet'
                    }
                `;

                layer.bindPopup(popupContent);

                // Add hover effects
                layer.on('mouseover', function(e) {
                    var layer = e.target;
                    layer.setStyle({
                        weight: 3,
                        color: '#666',
                        fillOpacity: 0.9
                    });
                });

                layer.on('mouseout', function(e) {
                    var layer = e.target;
                    layer.setStyle({
                        weight: 1,
                        color: 'white',
                        fillOpacity: getCountryOpacity(countryName)
                    });
                });
            }
        }).addTo(map);
    })
    .catch(error => {
        console.error('Error loading country data:', error);
        // Fallback: show message if GeoJSON fails to load
        document.getElementById('coverage-map').innerHTML =
            '<p style="text-align: center; padding: 50px;">Map loading... If this persists, please refresh the page.</p>';
    });

// Function to calculate centroid of a polygon
function getCentroid(coords, type) {
    if (type === 'Polygon') {
        return getPolygonCentroid(coords[0]);
    } else if (type === 'MultiPolygon') {
        // Use the largest polygon for centroid calculation
        var largestPolygon = coords.reduce((largest, current) => {
            return getPolygonArea(current[0]) > getPolygonArea(largest[0]) ? current : largest;
        });
        return getPolygonCentroid(largestPolygon[0]);
    }
    return null;
}

function getPolygonCentroid(coords) {
    var totalArea = 0;
    var centroidX = 0;
    var centroidY = 0;

    for (var i = 0; i < coords.length - 1; i++) {
        var x1 = coords[i][0];
        var y1 = coords[i][1];
        var x2 = coords[i + 1][0];
        var y2 = coords[i + 1][1];

        var area = (x1 * y2 - x2 * y1);
        totalArea += area;
        centroidX += (x1 + x2) * area;
        centroidY += (y1 + y2) * area;
    }

    totalArea /= 2;
    centroidX /= (6 * totalArea);
    centroidY /= (6 * totalArea);

    return [centroidY, centroidX]; // Return as [lat, lng]
}

function getPolygonArea(coords) {
    var area = 0;
    for (var i = 0; i < coords.length - 1; i++) {
        area += coords[i][0] * coords[i + 1][1] - coords[i + 1][0] * coords[i][1];
    }
    return Math.abs(area) / 2;
}

// Add a legend
var legend = L.control({position: 'bottomright'});

legend.onAdd = function (map) {
    var div = L.DomUtil.create('div', 'info legend');
    div.innerHTML = `
        <h4>Coverage Legend</h4>
        <div style="background: #90EE90; width: 20px; height: 20px; display: inline-block; margin-right: 5px; opacity: 0.7;"></div> Has Resources<br>
        <div style="background: #cccccc; width: 20px; height: 20px; display: inline-block; margin-right: 5px; opacity: 0.3;"></div> No Resources Yet<br>
        <small>Click countries for details</small>
    `;
    return div;
};

legend.addTo(map);

// Style the legend
var legendStyle = document.createElement('style');
legendStyle.textContent = `
    .info.legend {
        background: white;
        padding: 10px;
        border-radius: 5px;
        box-shadow: 0 0 15px rgba(0,0,0,0.2);
        font-size: 14px;
        max-width: 200px;
    }
    .info.legend h4 {
        margin: 0 0 10px 0;
        font-size: 16px;
    }
    .country-label {
        pointer-events: none;
    }
`;
document.head.appendChild(legendStyle);
</script>

<style>
#coverage-map {
    margin: 20px 0;
    height: 600px;
    width: 100%;
    border: 1px solid #ccc;
    border-radius: 8px;
    position: relative;
    z-index: 1;
}

body {
    margin: 0;
    padding: 0;
    overflow: visible;
}

@media (max-width: 768px) {
    #coverage-map {
        height: 400px;
    }
}
</style>
