<!-- Your Svelte component file -->
<script lang="ts">
    import { onMount } from 'svelte';
    import { Loader } from '@googlemaps/js-api-loader';

    const googleMapsApiKey = import.meta.env.VITE_GOOGLE_MAPS_API_KEY;
    const defaultPlace = {
        name: 'Rinconada Library',
        address: '1213 Newell Rd, Palo Alto, CA 94303',
    };
    let mapElement: HTMLElement;

    onMount(async () => {
        const loader = new Loader({
            apiKey: googleMapsApiKey,
            version: 'weekly', // Consider using a specific version for stability.
        });
        await loader.load();

        const geocoder = new google.maps.Geocoder();
        const geocoderResponse = await geocoder.geocode({ address: defaultPlace.address });
        const geocoderResult = geocoderResponse.results[0];
        const location = geocoderResult.geometry.location;
        
        const mapOptions = {
            center: location,
            zoom: zoom,
            tilt: 0,
            mapTypeId: 'satellite',
            mapTypeControl: false,
            fullscreenControl: false,
            rotateControl: false,
            streetViewControl: false,
            zoomControl: false,
            styles: [
                { // This style will hide the labels on the map
                    featureType: "all",
                    elementType: "labels",
                    stylers: [{ visibility: "off" }]
                }
            ],
        };

        const map = new google.maps.Map(mapElement, mapOptions);
    });
</script>

<!-- HTML structure -->
<div class="map-container" style="height: 100vh; width: 100vw;">
    <div bind:this={mapElement} />
</div>

<style>
    /* Ensure the map container takes up the full viewport size */
    .map-container {
        height: 100vh; /* 100% of the viewport height */
        width: 100vw; /* 100% of the viewport width */
    }
</style>

