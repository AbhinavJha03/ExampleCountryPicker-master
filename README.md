# ExampleCountryPicker
Android Country, State/Region, City picker




This is Android library for making Country, State/Region and City picker. When user select wished Country, State picker will automatically load States of picked Country, After that, when user pick State, City picker will automatically load Cities of selected State or Region.

In your applicatuin build gradle

implementation 'com.vikktorn.picker:picker:1.0.0'

Implement this listeners to your application activity implements OnStatePickerListener, OnCountryPickerListener, OnCityPickerListener

Make methode to get States and Cities from JSON from library

      public void getStateJson() throws JSONException {
      String json = null;
       try {
        InputStream inputStream = getAssets().open("states.json");
        int size = inputStream.available();
        byte[] buffer = new byte[size];
        inputStream.read(buffer);
        inputStream.close();
        json = new String(buffer, "UTF-8");

    } catch (IOException e) {
        e.printStackTrace();
    }


    JSONObject jsonObject = new JSONObject(json);
    JSONArray events = jsonObject.getJSONArray("states");
    for (int j = 0; j < events.length(); j++) {
        JSONObject cit = events.getJSONObject(j);
        State stateData = new State();

        stateData.setStateId(Integer.parseInt(cit.getString("id")));
        stateData.setStateName(cit.getString("name"));
        stateData.setCountryId(Integer.parseInt(cit.getString("country_id")));
        stateObject.add(stateData);
    }
}
// GET CITY FROM ASSETS JSON
public void getCityJson() throws JSONException {
    String json = null;
    try {
        InputStream inputStream = getAssets().open("cities.json");
        int size = inputStream.available();
        byte[] buffer = new byte[size];
        inputStream.read(buffer);
        inputStream.close();
        json = new String(buffer, "UTF-8");

    } catch (IOException e) {
        e.printStackTrace();
    }


    JSONObject jsonObject = new JSONObject(json);
    JSONArray events = jsonObject.getJSONArray("cities");
    for (int j = 0; j < events.length(); j++) {
        JSONObject cit = events.getJSONObject(j);
        City cityData = new City();

        cityData.setCityId(Integer.parseInt(cit.getString("id")));
        cityData.setCityName(cit.getString("name"));
        cityData.setStateId(Integer.parseInt(cit.getString("state_id")));
        cityObject.add(cityData);
    }
}
Create pickers

private CountryPicker countryPicker;
private StatePicker statePicker;
private CityPicker cityPicker;
Than set listeners on pickers

private void setListener() {
    pickStateButton.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            statePicker.showDialog(getSupportFragmentManager());
        }
    });
}
//SET COUNTRY LISTENER
private void setCountryListener() {
    pickCountry.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            countryPicker.showDialog(getSupportFragmentManager());
        }
    });
}
//SET CITY LISTENER
private void setCityListener() {
    pickCity.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            cityPicker.showDialog(getSupportFragmentManager());
        }
    });
}
And finale call methode onSelected from library

@Override
public void onSelectCountry(Country country) {
    // get country name and country ID
    countryName.setText(country.getName());
    countryID = country.getCountryId();
    statePicker.equalStateObject.clear();
    cityPicker.equalCityObject.clear();

    //set state name text view and state pick button invisible
    pickStateButton.setVisibility(View.VISIBLE);
    stateNameTextView.setVisibility(View.VISIBLE);
    stateNameTextView.setText("Region");
    cityName.setText("City");
    // set text on main view
    countryCode.setText("Country code: " + country.getCode());
    countryPhoneCode.setText("Country dial code: " + country.getDialCode());
    countryCurrency.setText("Country currency: " + country.getCurrency());
    flagImage.setBackgroundResource(country.getFlag());


    // GET STATES OF SELECTED COUNTRY
    for(int i = 0; i < stateObject.size(); i++) {
        // init state picker
        statePicker = new StatePicker.Builder().with(this).listener(this).build();
        State stateData = new State();
        if (stateObject.get(i).getCountryId() == countryID) {

            stateData.setStateId(stateObject.get(i).getStateId());
            stateData.setStateName(stateObject.get(i).getStateName());
            stateData.setCountryId(stateObject.get(i).getCountryId());
            stateData.setFlag(country.getFlag());
            statePicker.equalStateObject.add(stateData);
        }
    }
}
// ON SELECTED STATE ADD CITY TO PICKER
@Override
public void onSelectState(State state) {
    pickCity.setVisibility(View.VISIBLE);
    cityName.setVisibility(View.VISIBLE);
    cityName.setText("City");
    cityPicker.equalCityObject.clear();

    stateNameTextView.setText(state.getStateName());
    stateID = state.getStateId();



    for(int i = 0; i < cityObject.size(); i++) {
        cityPicker = new CityPicker.Builder().with(this).listener(this).build();
        City cityData = new City();
        if (cityObject.get(i).getStateId() == stateID) {
            cityData.setCityId(cityObject.get(i).getCityId());
            cityData.setCityName(cityObject.get(i).getCityName());
            cityData.setStateId(cityObject.get(i).getStateId());

            cityPicker.equalCityObject.add(cityData);
        }
    }
}
// ON SELECTED CITY
@Override
public void onSelectCity(City city) {

}
@Override
public void onPointerCaptureChanged(boolean hasCapture) {

}
Finally this is completly Java activity

public class MainActivity extends AppCompatActivity implements OnStatePickerListener, OnCountryPickerListener, OnCityPickerListener {

public static int countryID, stateID;
private Button pickStateButton, pickCountry, pickCity;
private TextView stateNameTextView, countryName,countryCode, countryPhoneCode, countryCurrency, cityName;
private ImageView flagImage;
// Pickers
private CountryPicker countryPicker;
private StatePicker statePicker;
private CityPicker cityPicker;
// arrays of state object
public static List<State> stateObject;
// arrays of city object
public static List<City> cityObject;



@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    // initialize view
    initView();
    // get state from assets JSON
    try {
        getStateJson();
    } catch (JSONException e) {
        e.printStackTrace();
    }
    // get City from assets JSON
    try {
        getCityJson();
    } catch (JSONException e) {
        e.printStackTrace();
    }

    // initialize country picker
    countryPicker = new CountryPicker.Builder().with(this).listener(this).build();

    // initialize listeners
    setListener();
    setCountryListener();
    setCityListener();

}




   public void initView(){
    //Buttons
    pickStateButton = (Button) findViewById(R.id.pickState);
    //set state picker invisible
    pickStateButton.setVisibility(View.INVISIBLE);
    pickCountry = (Button) findViewById(R.id.pickCountry);
    pickCity = (Button) findViewById(R.id.pick_city);
    // set city picker invisible
    pickCity.setVisibility(View.INVISIBLE);
    // Text Views
    countryName = (TextView) findViewById(R.id.countryNameTextView);
    countryCode = (TextView) findViewById(R.id.countryCodeTextView);
    countryPhoneCode = (TextView) findViewById(R.id.countryDialCodeTextView);
    countryCurrency = (TextView) findViewById(R.id.countryCurrencyTextView);
    stateNameTextView = (TextView) findViewById(R.id.state_name);
    //set state name text view invisible
    stateNameTextView.setVisibility(View.INVISIBLE);
    cityName = (TextView) findViewById(R.id.city_name);
    //set state name text view invisible
    cityName.setVisibility(View.INVISIBLE);

    // ImageView
    flagImage = (ImageView) findViewById(R.id.flag_image);

    // initiate state object, parser, and arrays
    stateObject = new ArrayList<>();
    cityObject = new ArrayList<>();
}

// SET STATE LISTENER
private void setListener() {
    pickStateButton.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            statePicker.showDialog(getSupportFragmentManager());
        }
    });
}
//SET COUNTRY LISTENER
private void setCountryListener() {
    pickCountry.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            countryPicker.showDialog(getSupportFragmentManager());
        }
    });
}
//SET CITY LISTENER
private void setCityListener() {
    pickCity.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            cityPicker.showDialog(getSupportFragmentManager());
        }
    });
}
// ON SELECTED COUNTRY ADD STATES TO PICKER
@Override
public void onSelectCountry(Country country) {
    // get country name and country ID
    countryName.setText(country.getName());
    countryID = country.getCountryId();
    statePicker.equalStateObject.clear();
    cityPicker.equalCityObject.clear();

    //set state name text view and state pick button invisible
    pickStateButton.setVisibility(View.VISIBLE);
    stateNameTextView.setVisibility(View.VISIBLE);
    stateNameTextView.setText("Region");
    cityName.setText("City");
    // set text on main view
    countryCode.setText("Country code: " + country.getCode());
    countryPhoneCode.setText("Country dial code: " + country.getDialCode());
    countryCurrency.setText("Country currency: " + country.getCurrency());
    flagImage.setBackgroundResource(country.getFlag());


    // GET STATES OF SELECTED COUNTRY
    for(int i = 0; i < stateObject.size(); i++) {
        // init state picker
        statePicker = new StatePicker.Builder().with(this).listener(this).build();
        State stateData = new State();
        if (stateObject.get(i).getCountryId() == countryID) {

            stateData.setStateId(stateObject.get(i).getStateId());
            stateData.setStateName(stateObject.get(i).getStateName());
            stateData.setCountryId(stateObject.get(i).getCountryId());
            stateData.setFlag(country.getFlag());
            statePicker.equalStateObject.add(stateData);
        }
    }
}
// ON SELECTED STATE ADD CITY TO PICKER
@Override
public void onSelectState(State state) {
    pickCity.setVisibility(View.VISIBLE);
    cityName.setVisibility(View.VISIBLE);
    cityName.setText("City");
    cityPicker.equalCityObject.clear();

    stateNameTextView.setText(state.getStateName());
    stateID = state.getStateId();



    for(int i = 0; i < cityObject.size(); i++) {
        cityPicker = new CityPicker.Builder().with(this).listener(this).build();
        City cityData = new City();
        if (cityObject.get(i).getStateId() == stateID) {
            cityData.setCityId(cityObject.get(i).getCityId());
            cityData.setCityName(cityObject.get(i).getCityName());
            cityData.setStateId(cityObject.get(i).getStateId());

            cityPicker.equalCityObject.add(cityData);
        }
    }
}
// ON SELECTED CITY
@Override
public void onSelectCity(City city) {
    cityName.setText(city.getCityName());
}
@Override
public void onPointerCaptureChanged(boolean hasCapture) {

}
// GET STATE FROM ASSETS JSON
public void getStateJson() throws JSONException {
    String json = null;
    try {
        InputStream inputStream = getAssets().open("states.json");
        int size = inputStream.available();
        byte[] buffer = new byte[size];
        inputStream.read(buffer);
        inputStream.close();
        json = new String(buffer, "UTF-8");

    } catch (IOException e) {
        e.printStackTrace();
    }


    JSONObject jsonObject = new JSONObject(json);
    JSONArray events = jsonObject.getJSONArray("states");
    for (int j = 0; j < events.length(); j++) {
        JSONObject cit = events.getJSONObject(j);
        State stateData = new State();

        stateData.setStateId(Integer.parseInt(cit.getString("id")));
        stateData.setStateName(cit.getString("name"));
        stateData.setCountryId(Integer.parseInt(cit.getString("country_id")));
        stateObject.add(stateData);
    }
}
// GET CITY FROM ASSETS JSON
public void getCityJson() throws JSONException {
    String json = null;
    try {
        InputStream inputStream = getAssets().open("cities.json");
        int size = inputStream.available();
        byte[] buffer = new byte[size];
        inputStream.read(buffer);
        inputStream.close();
        json = new String(buffer, "UTF-8");

    } catch (IOException e) {
        e.printStackTrace();
    }


    JSONObject jsonObject = new JSONObject(json);
    JSONArray events = jsonObject.getJSONArray("cities");
    for (int j = 0; j < events.length(); j++) {
        JSONObject cit = events.getJSONObject(j);
        City cityData = new City();

        cityData.setCityId(Integer.parseInt(cit.getString("id")));
        cityData.setCityName(cit.getString("name"));
        cityData.setStateId(Integer.parseInt(cit.getString("state_id")));
        cityObject.add(cityData);
    }}}



Thanks to Mukesh Solanki, mukeshsolanki I get idea from his country picker from this link:https://github.com/mukeshsolanki/country-picker-android, and I use his country Object in my library . Also thanks to Rohit Kumar, hiiamrohit because I use his JSON data for States and Cities from this link: http://www.iamrohit.in/countries-states-and-cities-database-of-world-in-mysql/

Also for some Countries I don't have list of States and Cities, so when State and City list is empty application break , but this is just for 10 Countries I think and I working on that.
