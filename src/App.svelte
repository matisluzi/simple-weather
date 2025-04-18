<script>
	import { Circle } from "svelte-loading-spinners";

	// import api key
	const API_KEY = import.meta.env.VITE_API_KEY;

	let errorString = $state(""); // error state
	let loading = $state(false); // loading state
	let cityInput = $state(""); // input value
	let units = $state("metric"); // units for temperature

	let city = $state(null); // city data from API
	let weather = $state(null); // weather data from API

	// fetch cities from API using input value
	async function getCities() {
		errorString = ""; // reset error state
		try {
			const response = await fetch(
				"http://dataservice.accuweather.com/locations/v1/cities/search?apikey=" +
					API_KEY +
					"&q=" +
					cityInput
			);
			if (!response.ok) {
				throw new Error("Response status: " + response.status);
			}
			const data = await response.json();
			if (data.length > 0) {
				city = data[0];
			} else {
				throw new Error("No city found");
			}
		} catch (error) {
			errorString = "Error loading city data: " + error.message; // set error state
			throw error; // rethrow error to break promise chain in
		}
	}

	// fetch weather data from API using city key
	async function getWeather() {
		errorString = ""; // reset error state
		if (city) {
			try {
				const response = await fetch(
					"http://dataservice.accuweather.com/currentconditions/v1/" +
						city.Key +
						"?apikey=" +
						API_KEY
				);
				if (!response.ok) {
					throw new Error("Response status: " + response.status);
				}
				const data = await response.json();
				if (data.length > 0) {
					weather = data[0];
				} else {
					throw new Error("No weather data found");
				}
			} catch (error) {
				errorString = "Error loading weather data: " + error.message; // set error state
			}
		} else {
			errorString = "Please enter a valid city name."; // set error state
		}
	}

	// handle form submission
	function handleFormSubmit(event) {
		// prevent default form submission
		event.preventDefault();

		loading = true; // set loading state
		errorString = ""; // reset error state

		getCities()
			.then(() => {
				return getWeather();
			})
			.catch((error) => {
				// handle error
				console.error("Error:", error);
				// error message is already set in getCities or getWeather
			})
			.finally(() => {
				loading = false; // reset loading state
			});
	}
</script>

<main class="container flex flex-col min-h-screen gap-4 p-4 md:mx-auto md:p-8">
	<!-- Header	-->
	<div class="flex flex-col gap-1">
		<h1 class="text-2xl font-bold">Simple Weather App</h1>
		<p class="text-sm text-secondary">
			Developed by Matis Luzi for Simple Syllabus
		</p>
	</div>

	<!-- Main content -->
	<div class="flex flex-col justify-center gap-4 grow">
		<!-- User input -->
		<form class="flex items-center gap-2" onsubmit={handleFormSubmit}>
			<input
				type="text"
				placeholder="Enter city name"
				bind:value={cityInput}
				class="w-full px-3 py-2 border rounded-lg bg-secondary md:max-w-48 border-neutral-700"
				onkeydown={(e) => {
					if (e.key === "Enter") {
						handleFormSubmit(e);
					}
				}}
				required
			/>
			<input
				class="flex items-center gap-1 px-3 py-2 transition-colors bg-indigo-700 rounded-lg hover:bg-indigo-700/70 active:bg-indigo-700/40 h-fit"
				type="submit"
				value={loading ? "Loading..." : "Get Weather"}
			/>
		</form>

		<!-- Weather Data -->
		<div class="relative flex flex-col gap-2 p-4 rounded-xl bg-secondary">
			{#if errorString != ""}
				<p class="text-red-500">
					{errorString}
				</p>
			{:else if weather && !loading}
				<button
					class="px-2 py-1 transition-colors bg-indigo-700 rounded-full md:absolute hover:bg-indigo-700/70 active:bg-indigo-700/40 top-4 right-4"
					onclick={() => {
						units = units === "metric" ? "imperial" : "metric";
					}}
				>
					{units === "metric" ? "°C" : "°F"}
				</button>
				<div class="flex flex-col md:flex-row">
					<h2 class="text-2xl font-bold">
						Weather in {city.LocalizedName}, {city.AdministrativeArea
							.CountryID === "US"
							? city.AdministrativeArea.LocalizedName
							: city.Country.LocalizedName}
					</h2>
					<img
						src={`https://developer.accuweather.com/sites/default/files/${
							weather.WeatherIcon < 10
								? "0" + weather.WeatherIcon
								: weather.WeatherIcon
						}-s.png`}
						alt="Weather Icon"
						class="object-contain object-left w-16 h-8"
					/>
				</div>

				<div class="flex flex-col gap-2 my-4">
					<p>Weather: {weather.WeatherText}</p>
					<p>
						Temperature: {units === "metric"
							? weather.Temperature.Metric.Value + "°C"
							: weather.Temperature.Imperial.Value + "°F"}
					</p>
				</div>
				<p class="text-sm text-secondary">
					Latest Update: {new Date(
						weather.LocalObservationDateTime
					).toLocaleTimeString("en-US", {
						hour: "2-digit",
						minute: "2-digit",
					})}
				</p>
			{:else if loading}
				<Circle size="40" color="white" />
			{:else}
				<p class="text-secondary">
					No weather data available. Please provide a city name.
				</p>
			{/if}
		</div>
	</div>
</main>
