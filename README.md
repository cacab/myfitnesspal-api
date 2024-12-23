# myfitnesspal-api

Library for accessing MyFitnessPal data.

## Usage

### Create a new session

```java
// Your custom implementation to log in and get session cookies
LoginHandler loginHandler = ...;
IMFPSession session = MFPSession.create(user, password, loginHandler);

// You can also delegate credentials to the LoginHandler and avoid passing them
IMFPSession session = MFPSession.create(loginHandler);
```

#### LoginHandler

Since `v0.4` you have to provide a `LoginHandler`
[implementation](src/main/java/com/gmail/marcosav2010/myfitnesspal/api/LoginHandler.java)
to get the session cookies since MFP added a captcha and broke the previous way of logging in.

~~Check out [this example Selenium implementation](src/test/java/com/gmail/marcosav2010/myfitnesspal/api/SeleniumLoginHandler.java)
used for some basic tests.~~

**Update**: MyFitnessPal has improved its capcha verification and some other
obstacles that made the Selenium approach not viable anymore.

I have made a [MyFitnessPal API Server](https://github.com/marcosav/myfitnesspal-api-server) which makes use of this
library and provides a REST API to access MFP data using browser cookies.
It also provides a python library alternative implementation in case this one stopped working.
But definitely there is no way to use this library as it was before.

### Accessing to diary

```java
Diary diary = session.toDiary();
```

### Some examples

```java
// Get one specific day data (to fetch specific data keep reading, this way can take more time and get unnecessary data)
Day day = diary.getFullDay(date);

// Or multiple days...
List<Day> days = diary.getFullDayRange(dates);

// You can specify which data you want to fetch (WATER, FOOD_NOTES, EXERCISE_NOTES, EXERCISE or FOOD)
day = diary.getDay(date, Diary.FOOD, Diary.FOOD_NOTES, Diary.WATER);

// Same applies for multi-day queries
days = diary.getDayRange(dates, Diary.EXERCISE, Diary.EXERCISE_NOTES);

// Fetch water amount or food notes individually
int water = diary.getWater(date);
String notes = diary.getFoodNotes(date);

// Do the same thing with (adding Diary.FOOD_NOTES and Diary.WATER)
water = day.getWater();
notes = day.getFoodNotes();

// Make sure you set FOOD parameter when fetching in order to do the following...
// Get day meals
DiaryMeal dinner = day.getMeals().get(2);
String name = dinner.getName(); // "Dinner" or whatever your third meal name is

// Get day nutrients
float carbs = dinner.get(FoodValues.CARBOHYDRATES);
float calcium = dinner.get(FoodValues.CALCIUM);

// Get food info
dinner.getFood().forEach(food -> {
  String name = food.getName();
  String brand = food.getBrand();
  String unit = food.getUnit();
  float amnt = food.getAmount();
  float energy = food.getEnergy();

  // You can also fetch food nutrients
  float protein = food.get(FoodValues.PROTEIN);
  ...
});
```

### Exercise data

Use `Day` to check fetched exercise data too:

```java
// Make sure you set EXERCISE parameter when fetching...
Day day = diary.getDay(date, Diary.EXERCISE);

// Check exercise notes (adding Diary.EXERCISE_NOTES on fetch)
String exerciseNotes = day.getExerciseNotes();

List<CardioExercise> cardioExercises = day.getCardioExercises();
List<StrengthExercise> strengthExercises = day.getStrengthExercises();

day.getCardioExercises().forEach(cardioExercise -> {
    String name = cardioExercise.getName();
    float energy = cardioExercise.getEnergy();
    int duration = cardioExercise.getDuration();
    ...
});

day.getStrengthExercises().forEach(strengthExercise -> {
    String name = strengthExercise.getName();
    int quantity = strengthExercise.getQuantity(); // weight * repetitions
    int sets = strengthExercise.getSets();
    int repetitions = strengthExercise.getRepetitions();
    float weight = strengthExercise.getWeight();
    ...
});
```

### User data and settings

Check user data and settings using `UserData`:

```java
UserData userData = session.toUser();

String energyUnit = userData.getUnit(Unit.ENERGY); // calories
String weightUnit = userData.getUnit(Unit.WEIGHT); // kg

List<String> meals = userData.getMealNames(); // ["Breakfast", "Launch"...]
List<String> trackedNutrients = userData.getTrackedNutrients(); // ["carbohydrates", "fat"...]

String email = userData.getEmail();
...
```

### Notes for Sycing with google sheets
