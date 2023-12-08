<!--
 Copyright 2023 Google LLC

 Licensed under the Apache License, Version 2.0 (the "License");
 you may not use this file except in compliance with the License.
 You may obtain a copy of the License at

      https://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing, software
 distributed under the License is distributed on an "AS IS" BASIS,
 WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 See the License for the specific language governing permissions and
 limitations under the License.
 -->
<script lang="ts">
  import { onMount } from 'svelte';
  import type { MdFilledTextField } from '@material/web/textfield/filled-text-field';

  export let placesLibrary: google.maps.places.PlacesService;
  export let map: google.maps.Map;
  export let initialValue = '';
  export let zoom = 19;

  let textFieldElement: MdFilledTextField;
  let marker: google.maps.Marker;
  let geocoder = new google.maps.Geocoder();
  
  const defaultLocation = {lat: -34.397, lng: 150.644}; // Default fallback location coordinates

  onMount(async () => {
    await textFieldElement.updateComplete;
    const inputElement = textFieldElement.renderRoot.querySelector('input') as HTMLInputElement;

    const autocomplete = new google.maps.places.Autocomplete(inputElement, {
      fields: ['formatted_address', 'geometry', 'name'],
    });

    autocomplete.addListener('place_changed', () => {
      const place = autocomplete.getPlace();
      if (!place.geometry || !place.geometry.location) {
        console.error('No geometry found for this place.');
        textFieldElement.value = '';
        return;
      }
      updateMapAndMarker(place.geometry.location);
      updateTextField(place);
    });

    // Set the initial position for the marker
    const initialPosition = defaultLocation;

    marker = new google.maps.Marker({
      position: initialPosition,
      map: map,
      draggable: true,
      visible: true  // Make sure the marker is visible
    });

    // Initiate the map and marker to the initial position
    updateMapAndMarker(initialPosition);

    marker.addListener('dragend', () => {
      const newPosition = marker.getPosition();
      updateMapAndMarker(newPosition);
      reverseGeocodeAndUpdateTextField(newPosition);
    });

    map.addListener('click', (e) => {
      const clickPosition = e.latLng;
      marker.setPosition(clickPosition);
      updateMapAndMarker(clickPosition);
      reverseGeocodeAndUpdateTextField(clickPosition);
    });

    // Log marker and map status for debugging
    console.log('Marker initialized at:', initialPosition);
    console.log('Map center set to:', initialPosition);
  });

  function updateMapAndMarker(location) {
    map.setCenter(location);
    map.setZoom(zoom);
    marker.setPosition(location);
    console.log('Map and marker updated to:', location);  // Log for debugging
  }

  function updateTextField(place) {
    textFieldElement.value = place.name || place.formatted_address || '';
  }

  async function reverseGeocodeAndUpdateTextField(location) {
    try {
      const results = await geocoder.geocode({ location: location });
      if (results.results[0]) {
        updateTextField({ name: '', formatted_address: results.results[0].formatted_address });
      } else {
        textFieldElement.value = 'No address found';
      }
    } catch (e) {
      textFieldElement.value = 'Geocoding failed';
      console.error('Geocoding failed:', e);  // Log for debugging
    }
  }
</script>

<md-filled-text-field bind:this={textFieldElement} label="Search an address" />
