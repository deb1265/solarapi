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
  /* global google */
  import { onMount } from 'svelte';
  import type { MdFilledTextField } from '@material/web/textfield/filled-text-field';

  export let placesLibrary: google.maps.PlacesLibrary;
  export let map: google.maps.Map;
  export let initialValue = '';
  export let zoom = 19;

  let textFieldElement: MdFilledTextField;
  let marker;

  onMount(async () => {
    await textFieldElement.updateComplete;
    const inputElement = textFieldElement.renderRoot.querySelector('input') as HTMLInputElement;

    // Initialize Autocomplete
    const autocomplete = new placesLibrary.Autocomplete(inputElement, {
      fields: ['formatted_address', 'geometry', 'name'],
    });
    
    autocomplete.addListener('place_changed', () => {
      const place = autocomplete.getPlace();
      if (!place.geometry || !place.geometry.location) {
        textFieldElement.value = '';
        return;
      }
      updateMapAndMarker(place.geometry.location);
      updateTextField(place);
    });

    // Initialize Marker
    marker = new google.maps.Marker({
      map: map,
      draggable: true
    });

    // Marker dragend event
    marker.addListener('dragend', () => {
      const newPosition = marker.getPosition();
      updateMapAndMarker(newPosition);
      reverseGeocodeAndUpdateTextField(newPosition);
    });

    // Map click event
    map.addListener('click', (e) => {
      marker.setPosition(e.latLng);
      updateMapAndMarker(e.latLng);
      reverseGeocodeAndUpdateTextField(e.latLng);
    });
  });

  function updateMapAndMarker(location) {
    map.setCenter(location);
    map.setZoom(zoom);
    marker.setPosition(location);
  }

  function updateTextField(place) {
    textFieldElement.value = place.name || place.formatted_address || '';
  }

  async function reverseGeocodeAndUpdateTextField(location) {
    try {
      const geocoder = new google.maps.Geocoder();
      const results = await geocoder.geocode({ location: location });
      if (results.results[0]) {
        updateTextField({ name: '', formatted_address: results.results[0].formatted_address });
      } else {
        textFieldElement.value = 'No address found';
      }
    } catch (e) {
      textFieldElement.value = 'Geocoding failed';
    }
  }
</script>

<md-filled-text-field bind:this={textFieldElement} label="Search an address" value={initialValue}>
  <md-icon slot="leadingicon">search</md-icon>
</md-filled-text-field>

