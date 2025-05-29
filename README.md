@page "/"
@rendermode InteractiveServer
@using System.Net.Http.Json
@inject HttpClient Http
@inject IJSRuntime JS

<h3>Bank Search</h3>

<div class="form-group">
  <label for="cityInput">Enter location or city:</label>
  <input type="text" class="form-control" id="cityInput" @bind="SearchQuery" placeholder="e.g. Praha" />
  <button class="btn btn-primary mt-2" @onclick="SearchBanks">Search Banks</button>
</div>

<div id="map" style="height:500px;width:100%;margin-top:20px;"></div>

@if (Results?.Count > 0)
{
    <div class="container">
        <h5 class="text-center mb-4">Results:</h5>
        <div class="row g-4">
            @foreach (var result in Results)
            {
                <div class="col-md-4">
                    <div class="result-card p-3 h-100">
                        <strong>@result.Poi.Name</strong>
                        <small class="d-block mt-1">@result.Address.FreeformAddress</small>
                    </div>
                </div>
            }
        </div>
    </div>
}


@code {
    private string SearchQuery = string.Empty;
    private List<SearchResult> Results;
  
    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            // Initialize the map with a default center
            await JS.InvokeVoidAsync("initializeMap", new List<SearchResult>());
        }
    }

    private async Task SearchBanks()
    {
        if (string.IsNullOrWhiteSpace(SearchQuery)) return;

        var subscriptionKey = "9L98ONwzVRUxJoCBRC0eS7LsA2RnB7ghI0qZfeIt5aVCRZzDykbFJQQJ99BEAC5RqLJZjxS7AAAgAZMP2QkN";
        var apiUrl = $"https://atlas.microsoft.com/search/fuzzy/json?api-version=1.0&subscription-key={subscriptionKey}&query=bank {Uri.EscapeDataString(SearchQuery)}&language=cs-CZ&limit=10";

        var response = await Http.GetFromJsonAsync<AzureMapsResponse>(apiUrl);
        Results = response?.Results ?? new();

        // Update the map with search results after fetching
        await JS.InvokeVoidAsync("initializeMap", Results);
    }

    public class AzureMapsResponse
    {
        public List<SearchResult> Results { get; set; }
    }

    public class SearchResult
    {
        public Poi Poi { get; set; }
        public Address Address { get; set; }
        public Position Position { get; set; }
    }

    public class Poi
    {
        public string Name { get; set; }
    }

    public class Address
    {
        public string FreeformAddress { get; set; }
    }

    public class Position
    {
        public double Lat { get; set; }
        public double Lon { get; set; }
    }
}


function initializeMap(results) {
    // Set default center if no results are passed
    const defaultCenter = [14.4208, 50.0880]; // Coordinates for Prague

    var map = new atlas.Map('map', {
        center: defaultCenter,
        zoom: 2,
        authOptions: {
            authType: 'subscriptionKey',
            subscriptionKey: '9L98ONwzVRUxJoCBRC0eS7LsA2RnB7ghI0qZfeIt5aVCRZzDykbFJQQJ99BEAC5RqLJZjxS7AAAgAZMP2QkN'
        }
    });

    // If there are results, add markers
    if (results && results.length > 0) {
        results.forEach(r => {
            map.markers.add(new atlas.HtmlMarker({
                position: [r.position.lon, r.position.lat],
                htmlContent: '<div style="background:#FFCC00;width:20px;height:20px;border-radius:50%;border:2px solid #002E3C;"></div>'
            }));
            map.setCamera({center: [r.position.lon, r.position.lat], zoom: 10});
        });
    } else {
        // If no results, zoom out to show the whole world or a default location
        map.setCamera({center: defaultCenter, zoom: 2});
    }
}
