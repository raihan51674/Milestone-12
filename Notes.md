#### Module -1
### 1.Navber create -responsive
### 2.Banner create use - Carosol or swiper (react libary)
```js
1.add react-responsive-carousel
2. npm install react-fast-marquee
3. npm i swiper
```
### 3.AOS use each website must be 
```js
1. npm install aos
2. main.jsx file add :
import AOS from 'aos';
import 'aos/dist/aos.css';
AOS.init();
3. use any component :
<div data-aos="fade-right"></div>


```

#### Module -2
### 4.Leaflet use map integration
```js
// i need create a page called coverage there will have a title we are availabe in 64 districts below it will be a search box where i can search name of different district in Bangladesh (will tell you detail about search box later)
// First give me a map that i can show in the website that will show now i need to add explain the code details

import React from 'react';
import BangladeshMap from './BangladeshMap';
import { useLoaderData } from 'react-router';


const Coverage = () => {
    const serviceCenters = useLoaderData();
    
    return (
        <div className="max-w-4xl mx-auto px-4 py-10">
            <h1 className="text-3xl font-bold text-center mb-6">We are available in 64 districts</h1>

            {/* Later you can add your search box here */}
            {/* <SearchDistrictBox /> */}

            <BangladeshMap serviceCenters={serviceCenters} />
        </div>
    );
};

export default Coverage;


import { MapContainer, TileLayer, Marker, Popup, useMap } from 'react-leaflet';
import L from 'leaflet';
import 'leaflet/dist/leaflet.css';
import { useState } from 'react';

const position = [23.6850, 90.3563]; // Center of Bangladesh

// Optional custom icon (can skip for default)
const customIcon = new L.Icon({
    iconUrl: 'https://unpkg.com/leaflet@1.9.4/dist/images/marker-icon.png',
    iconSize: [25, 41],
    iconAnchor: [12, 41],
});

// Helper component to move map
function FlyToDistrict({ coords }) {
    const map = useMap();
    if (coords) {
        map.flyTo(coords, 14, { duration: 1.5 });
    }
    return null;
}

const BangladeshMap = ({ serviceCenters }) => {
    const [searchText, setSearchText] = useState('');
    const [activeCoords, setActiveCoords] = useState(null);
    const [activeDistrict, setActiveDistrict] = useState(null);

    const handleSearch = (e) => {
        e.preventDefault();
        const district = serviceCenters.find(d =>
            d.district.toLowerCase().includes(searchText.toLowerCase())
        );
        if (district) {
            setActiveCoords([district.latitude, district.longitude]);
            setActiveDistrict(district.district);
        }
    };

    return (
        <div className="h-[800px] w-full rounded-lg overflow-hidden shadow-lg relative">

            <form
                onSubmit={handleSearch}
                className="absolute top-4 left-1/2 transform -translate-x-1/2 z-[1000] w-full max-w-md px-4 flex bg-gray-400"
            >
                <input
                    type="text"
                    placeholder="Search district..."
                    className="flex-1 px-4 py-2 border rounded-l-md outline-none"
                    value={searchText}
                    onChange={(e) => setSearchText(e.target.value)}
                />
                <button
                    type="submit"
                    className="bg-blue-600 text-white px-4 py-2 rounded-r-md hover:bg-blue-700"
                >
                    Go
                </button>
            </form>


            {/* map container */}
            <MapContainer center={position} zoom={8} scrollWheelZoom={false} className="h-full w-full z-0">
                <TileLayer
                    attribution='&copy; <a href="http://osm.org/copyright">OpenStreetMap</a>'
                    url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
                />

                <FlyToDistrict coords={activeCoords} />

                {
                    serviceCenters.map((center, index) => <Marker
                        key={index}
                        position={[center.latitude, center.longitude]}
                        icon={customIcon}>
                        <Popup autoOpen={center.district === activeDistrict}>
                            <strong>{center.district}</strong><br />
                            {center.covered_area.join(', ')}
                        </Popup>
                    </Marker>)
                }
            </MapContainer>
        </div>
    );
};

export default BangladeshMap;
```
### 5.
