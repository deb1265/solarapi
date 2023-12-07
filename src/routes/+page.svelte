<!-- Your Svelte component file -->
<script lang="ts">
    import { onMount } from 'svelte';
    import { Loader } from '@googlemaps/js-api-loader';
    import SearchBar from './components/SearchBar.svelte';
    import Sections from './sections/Sections.svelte';

    const googleMapsApiKey = import.meta.env.VITE_GOOGLE_MAPS_API_KEY;
    const defaultPlace = {
        name: 'Rinconada Library',
        address: '1213 Newell Rd, Palo Alto, CA 94303',
    };
    let location: google.maps.LatLng | undefined;
    const zoom = 19;
    let mapElement: HTMLElement;
    let map: google.maps.Map;
    let geometryLibrary: google.maps.GeometryLibrary;
    let mapsLibrary: google.maps.MapsLibrary;
    let placesLibrary: google.maps.PlacesLibrary;

    onMount(async () => {
        const loader = new Loader({ apiKey: googleMapsApiKey });
        const libraries = {
            geometry: loader.importLibrary('geometry'),
            maps: loader.importLibrary('maps'),
            places: loader.importLibrary('places'),
        };
        geometryLibrary = await libraries.geometry;
        mapsLibrary = await libraries.maps;
        placesLibrary = await libraries.places;

        const geocoder = new google.maps.Geocoder();
        const geocoderResponse = await geocoder.geocode({ address: defaultPlace.address });
        const geocoderResult = geocoderResponse.results[0];
        location = geocoderResult.geometry.location;
        map = new mapsLibrary.Map(mapElement, {
            center: location,
            zoom: zoom,
            tilt: 0,
            mapTypeId: 'satellite',
            mapTypeControl: false,
            fullscreenControl: false,
            rotateControl: false,
            streetViewControl: false,
            zoomControl: false,
        });

        // Load HD image
        await loadHDImageOfAddress(defaultPlace.address);
    });

    // Function to load HD image of a given address
    async function loadHDImageOfAddress(address: string) {
        const geocoder = new google.maps.Geocoder();
        const geocoderResponse = await geocoder.geocode({ address: address });
        const geocoderResult = geocoderResponse.results[0];
        const location = geocoderResult.geometry.location;

        const imageUrl = `https://maps.googleapis.com/maps/api/staticmap?center=${location.lat()},${location.lng()}&zoom=15&size=600x300&scale=2&key=${googleMapsApiKey}`;
        const imageElement = document.getElementById('hdImage');
        if (imageElement) {
            imageElement.src = imageUrl;
        }
    }
</script>

<!-- HTML structure -->
<div class="flex flex-row h-full">
    <!-- Main map -->
    <div bind:this={mapElement} class="w-full" />

    <!-- Side bar -->
    <aside class="flex-none md:w-96 w-80 p-2 pt-3 overflow-auto">
        <div class="flex flex-col space-y-2 h-full">
            {#if placesLibrary && map}
                <SearchBar bind:location {placesLibrary} {map} initialValue={defaultPlace.name} />
            {/if}

            {#if location}
                <Sections {location} {map} {geometryLibrary} {googleMapsApiKey} />
            {/if}

            <div class="grow" />

            <div class="flex flex-col items-center w-full">
                <md-text-button
                    href="https://github.com/googlemaps-samples/js-solar-potential"
                    target="_blank"
                >
                    View code on GitHub
                    <img slot="icon" src="github-mark.svg" alt="GitHub" width="16" height="16" />
                </md-text-button>
            </div>

            <span class="pb-4 text-center outline-text label-small">
                This is not an officially supported Google product.
            </span>
        </div>
    </aside>
</div>

<!-- Element to display the HD image -->
<div class="hd-image-container">
    <img id="hdImage" alt="HD View" />
</div>

