<script>
  import ArticleHeader from '$lib/components/Article/ArticleHeader.svelte';
  import ArticleBody from '$lib/components/Article/ArticleBody.svelte';
  import Map from '$lib/components/Maps/Map.svelte';
  import MapLayer from '$lib/components/Maps/MapLayer.svelte';
  import Geocoder from '$lib/components/Maps/Geocoder.svelte';
  import Legend from '$lib/components/Maps/Legend.svelte';

  let { data } = $props();
  const preks = data.preks;
  console.log(preks);
  let longitude = $state(-73.97);
  let latitude = $state(40.7);
  let zoom = $state(12.5);
  let searchedLocation = $state(null);

  function toRadians(value) {
    return (value * Math.PI) / 180;
  }

  function distanceInMeters(startLatitude, startLongitude, endLatitude, endLongitude) {
    const earthRadiusMeters = 6371000;
    const latitudeDelta = toRadians(endLatitude - startLatitude);
    const longitudeDelta = toRadians(endLongitude - startLongitude);
    const startLatitudeRadians = toRadians(startLatitude);
    const endLatitudeRadians = toRadians(endLatitude);

    const a =
      Math.sin(latitudeDelta / 2) * Math.sin(latitudeDelta / 2) +
      Math.cos(startLatitudeRadians) *
        Math.cos(endLatitudeRadians) *
        Math.sin(longitudeDelta / 2) *
        Math.sin(longitudeDelta / 2);

    return 2 * earthRadiusMeters * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
  }

  const nearestPreschool = $derived.by(() => {
    if (!searchedLocation) return null;

    let nearestFeature = null;
    let shortestDistance = Infinity;

    for (const feature of preks.features ?? []) {
      const coordinates = feature?.geometry?.coordinates;
      if (!Array.isArray(coordinates) || coordinates.length < 2) continue;

      const [featureLongitude, featureLatitude] = coordinates;
      const currentDistance = distanceInMeters(
        searchedLocation.latitude,
        searchedLocation.longitude,
        featureLatitude,
        featureLongitude
      );

      if (currentDistance < shortestDistance) {
        shortestDistance = currentDistance;
        nearestFeature = feature;
      }
    }

    return nearestFeature;
  });
</script>

<div class="container">
  <ArticleHeader
    headline="Pre-K Locations In NYC"
    byline="NYCity News Service"
    pubDate="2026-04-26"
  />
<ArticleBody>
  <p>
  This is a map of pre-K locations in New York City. The city government promises a seat for every four-year-old in New York City. Pre-K is essential for helping children develop proper social skills and sets them up for a better path in life. The below map is color coded by the number of seats available at each school. The data is taken from NYC OpenData and the metadata was last updated in November 2024. 
  </p> 
  <br>
</ArticleBody>
    <Geocoder
    label="Find your neighborhood"
    placeholder="Enter an address in New York…"
    onresult={(result) => {
      searchedLocation = {
        latitude: result.lat,
        longitude: result.lng,
      };
      longitude = result.lng;
      latitude = result.lat;
      zoom = 15;
    }}
  />

  {#if nearestPreschool}
    <p class="nearest-preschool">
      The nearest preschool is {nearestPreschool.properties.name}.
    </p>
  {/if}

  <Map
    {longitude}
    {latitude}
    {zoom}
    height={600}
    theme="positron"
    credit="OpenFreeMap / OpenStreetMap contributors"
  >
    <MapLayer
      id="pre-k"
      type="circle"
      data={preks}
      paint={{
        'circle-color': [
          'interpolate',
          ['linear'],
          ['to-number', ['get', 'seats'], 0],
          0,
          '#dbeafe',
          50,
          '#93c5fd',
          100,
          '#3b82f6',
          150,
          '#1d4ed8',
          235,
          '#1e3a8a',
        ],
        'circle-radius': 5,
        'circle-stroke-color': '#1a1a1a',
        'circle-stroke-width': 1,
        'circle-opacity': 0.9,
      }}
      popup={(feature) => {
        const p = feature.properties;
        return `<strong>${p.name}</strong><br>${p.address}<br>This school has <strong> ${p.seats} </strong> seats`;
      }}
    />
  </Map>

    <Legend
    title="Pre-Schools by seat number"
    mode="threshold"
    items={[
      {
        color: '#dbeafe',
        to: 50,
      },
      {
        color: '#93c5fd',
        from: 51,
        to: 100,
      },
      {
        color: '#3b82f6',
        from: 101,
        to: 150,
      },
      {
        color: '#1d4ed8',
        from: 151,
        to: 235,
      },
      {
        color: '#1e3a8a',
        from: 236,
      },
    ]}
  />
</div>

<style lang="scss">
  .nearest-preschool {
    margin-top: var(--spacing-xs);
    margin-bottom: var(--spacing-sm);
    font-size: var(--font-size-md);
    font-weight: var(--font-weight-semibold);
  }
</style>