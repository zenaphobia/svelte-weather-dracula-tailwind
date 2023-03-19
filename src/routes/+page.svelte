<script lang="ts">
    import { City, type ICity } from 'country-state-city'

    const allStates = City.getCitiesOfCountry('US')
    let data:Promise<JSON | undefined> | JSON | undefined
    let selectedCity:ICity | undefined
    let input:string = ""
    let filteredCities: ICity[] | undefined

    //Getting a six day forecast
    $: forecast = data?.daily.time.splice(1)

    $: selectedCity = undefined
    $: filteredCities = input ? allStates?.filter(p => p.name.toLowerCase().includes(input.toLowerCase())).slice(0, filteredCities?.length - (filteredCities?.length - 20)) : undefined

    async function handleClick(_city:ICity) {
        selectedCity = _city
        input = _city.name + ',' + ' ' + _city.stateCode
        data = await getWeather(selectedCity)
    }

    function getWeekdayText(data:string){
        let _data
        let _input = data + 'T22:00';
        _data = new Date(_input).toLocaleDateString("en-US",{weekday:'long', timeZone:'America/Los_Angeles'})
        return _data;
    }

    async function getWeather(params:ICity):Promise<JSON | undefined>{
        const long = params.longitude
        const lat = params.latitude
        const url = `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${long}&daily=weathercode&daily=temperature_2m_max,temperature_2m_min&current_weather=true&temperature_unit=fahrenheit&windspeed_unit=mph&precipitation_unit=inch&timezone=America%2FLos_Angeles`
        console.log(url)
        const response = await fetch(url)
        let _data = undefined

        if(response.ok){
            _data = await response.json()
            return _data
        }

        return _data
    }

    function getWeatherCode(_weather:number){
        switch(_weather){
            case 0:
                return "https://img.icons8.com/FFFFFF/ios-glyphs/100/null/sun--v1.png"
            case 1:
            case 2:
            case 3:
                return "https://img.icons8.com/FFFFFF/ios-glyphs/100/null/partly-cloudy-day--v1.png"
            case 51:
            case 53:
            case 55:
                return "https://img.icons8.com/FFFFFF/ios-glyphs/100/null/light-rain.png"
            case 61:
            case 63:
            case 65:
                return "https://img.icons8.com/FFFFFF/ios-glyphs/100/null/downpour.png"
            case 71:
                return "https://img.icons8.com/FFFFFF/ios-glyphs/100/null/light-snow.png"
            case 73:
            case 75:
                return "https://img.icons8.com/FFFFFF/ios-filled/100/null/snow-storm.png"
            case 80:
            case 81:
            case 82:
                return "https://img.icons8.com/FFFFFF/ios-filled/100/null/intense-rain.png"
            case 95:
                return "https://img.icons8.com/FFFFFF/ios-glyphs/100/null/cloud-lighting--v1.png"
            default:
                return ""

        }
    }

    function getWeatherText(_weather:number){
        switch(_weather){
            case 0:
                return "Clear"
            case 1:
                return "Mainly clear"
            case 2:
                return "Partly cloudy"
            case 3:
                return "Overcast"
            case 45:
                return "Fog"
            case 48:
                return "Depositing rime fog"
            case 51:
                return "Light drizzle"
            case 53:
                return "Moderate drizzle"
            case 55:
                return "Dense drizzle"
            case 56:
                return "Light freezing drizzle"
            case 57:
                return "Dense freezing drizzle"
            case 61:
                return "Slight rain"
            case 63:
                return "Moderate rain"
            case 65:
                return "Heavy rain"
            case 71:
                return "Slight snow fall"
            case 73:
                return "Moderate snow fall"
            case 75:
                return "Heavy snow fall"
            case 80:
                return "Slight rain showers"
            case 81:
                return "Moderate rain showers"
            case 82:
                return "Violent rain showers"
            case 85:
                return "Slight snow showers"
            case 86:
                return "Heavy snow showers"
            case 95:
                return "Thunderstorm"
            case 96:
                return "Thunderstorm & slight hail"
            case 99:
                return "Thunderstorm & heavy hail"
            default:
                return "No text for this case"+_weather

        }
    }

    function resetForm(){
        input = ''
        selectedCity = undefined
        data = undefined
    }

</script>

<div class="drac-box container-full mx-auto drac-p-lg">
    <div class="drac-box drac-card max-w-7xl mx-auto drac-card-subtle drac-border-purple drac-p-md drac-m-md drac-p-sm" data-variant="subtle">
        <h1 class="drac-heading font-bold drac-heading-2xl drac-text-white">Quick Weather</h1>
        <p class="drac-text drac-line-height text-gray-400">Enter your city and get a quick 5 day forecast</p>
        <div class="drac-box drac-text-white max-w-full">
            <div class="drac-py-md box-border" style="position: relative">
                <div class="flex flex-row box-border p-2">
                    <p class="text text-gray-600 pr-2" style={input ? "opacity: .5" : "opacity: 1"}>Input city name</p>
                    <button class="box-border rounded hover:rounded-lg hover:bg-slate-100 bg-slate-400 text-slate-700 px-4" on:click={()=>{resetForm()}}>Reset</button>
                </div>
                <input class="drac-input drac-input-white drac-text-white" type="value" bind:value={input} name="" id="">
                <div class="drac-box list-component drac-bg-grey-secondary border-radius" style={filteredCities ? "max-height: 250px" : "max-height: 0px"}>
                    <ul class="drac-list list">
                        {#if filteredCities}
                            {#each filteredCities as city, i}
                                <li on:click={()=>{handleClick(city)}} on:keypress={()=>{handleClick(city)}} class="drac-text drac-text-white drac-my-xs list hover:bg-violet-500" value={city}> {city.name} <span class="font-bold text-white">[{city.stateCode}]</span></li>
                            {/each}
                        {/if}
                    </ul>
                </div>
            </div>
            <!-- <button class="drac-btn drac-my-sm drac-bg-white drac-text-black">See Data</button> -->
        </div>
        <div class="box-border flex justify-center md:justify-around flex-col md:flex-row columns-1 lg:columns-2 content-center items-stretch">
            {#if data}
                <div class=" bg-gradient-to-br from-indigo-500 mb-2 lg:m-4 box-border flex flex-col overflow-hidden fade-in justify-center items-center w-full lg:w-1/2 rounded-lg">
                    <div>
                        <h3 class="text-slate-400">Today</h3>
                    </div>
                    <div class="flex flex-row">
                        <div class="shrink-1 p-4">
                            <img src={getWeatherCode(data.current_weather.weathercode)} alt={getWeatherText(data.current_weather.weathercode)}>
                        </div>
                        <div class="p-4">
                            <h3 class="text-white text-3xl md:text-5xl lg:text-7xl font-bold">{data.current_weather.temperature}°F</h3>
                            <p class="text-slate-400 text-2xl">{getWeatherText(data.current_weather.weathercode)}</p>
                        </div>
                    </div>
                </div>
                <div class="box-border flex flex-row flex-wrap columns-1 lg:columns-2 lg:w-1/2 justify-around">
                    {#each forecast as day, i}
                        <div class="p-4 my-2 lg:m-4 flex flex-wrap box-border flex-col justify-center items-center w-full lg:w-36 bg-slate-800 rounded-lg">
                            <p class="text-white font-bold">{getWeekdayText(day)}</p>
                            <div>
                                <img class="w-12" src={getWeatherCode(data.daily.weathercode[i])} alt="">
                            </div>
                            <p class="text-white text-center">{getWeatherText(data.daily.weathercode[i])}</p>
                            <p class="text-slate-400">L: {data.daily.temperature_2m_min[i]}<span class="text-slate-500"> °F</span></p>
                            <p class="text-slate-400">H: {data.daily.temperature_2m_max[i]}<span class="text-slate-500"> °F</span></p>
                        </div>
                    {/each}
                </div>
            {/if}
        </div>
    </div>
</div>


<style>
    * {
        transition: all 0.3s ease-in-out;
    }
    @keyframes fade-in {
        from {opacity: 0;}
        to {opacity: 1;}
    }
    .fade-in {
        animation-name: fade-in;
        animation-duration: .5s;
        animation-timing-function: ease-in-out;
    }
    .border-radius {
        border-radius: 0 0 1rem 1rem;
        overflow: hidden;
    }
    .list-component {
        min-height: 0px;
        max-height: 250px;
        overflow-y: auto;
    }

    ul.list {
        padding: 0;
        margin: 0;
    }

    li.list {
        padding-left: 2rem;
        padding-right: 2rem;
        padding-top: 1rem;
        padding-bottom: 1rem;
        margin: 0;
        list-style-type: none;
    }

    ul > li.list:nth-child(odd) {
        background-color: drac-bg-grey-secondary;
    }

    li:hover {
        cursor: pointer;
    }
</style>