
## Managing App-Wide States

### React Context

Primarily used for ```prop drilling``` solutions, making ```data available throughout the component tree``` without having to pass props manually at every level. Suitable for simple state management scenarios such as themes, user authentication data, or language settings.

```
my-project/
│...
├── store/
│ ├── context/
│ ├── services/
|...
├── App.tsx/
```

#### ../store/context/favorites-context.ts
```ts
import { createContext, useState } from 'react';

export const FavoritesContext = createContext({
  ids: [],
  addFavorite: (id) => {},
  removeFavorite: (id) => {}
});

function FavoritesContextProvider({children}) {
  const [favorites, setFavorites] = useState([]);
  ...

  function xAddFavorite(id) {
    setFavorites((currentFavorites) => [...currentFavorites, id]);
  }

  function xRemoveFavorite(id) {
    setFavorites((currentFavorites) => 
        currentFavorites.filter((favId) => favId !== id)
    );
  }
  ...

  const myValue = {
    ids: favorites,
    addFavorite: xAddFavorite,
    removeFavorite: xRemoveFavorite,
  }

  return <FavoritesContext.Provider value={myValue}>{children}</FavoritesContext.Provider>
}

export default FavoritesContextProvider;
```

#### ../App.tsx
```ts
...
import { FavoritesContextProvider } from '../store/context/favorites-context';
...
export default function App() {
  return (
    <>
      <StatusBar style="light" />
      <FavoritesContextProvider>
        <NavigationContainer>
          <Stack.Navigator
            screenOptions={{
              ...
            }}
          >
            <Stack.Screen
              name="Drawer"
              component={DrawerNavigator}
              options={{
                headerShown: false,
                ...
              }}
            />
            ...
          </Stack.Navigator>
        </NavigationContainer>
      </FavoritesContextProvider>
    </>
  );
}
...
```

#### ../screens/MealDetails.tsx
```ts
...
import { FavoritesContext } from '../store/context/favorites-context';
...
function MealDetailScreen({ route, navigation }) {
  const favoriteMealsCtx = useContext(FavoritesContext);

  const mealId = route.params.mealId;
  const selectedMeal = MEALS.find((meal) => meal.id === mealId);

  const mealIsFavorite = favoriteMealsCtx.ids.includes(mealId);

  function changeFavoriteStatusHandler() {
    if (mealIsFavorite) {
      favoriteMealsCtx.removeFavorite(mealId);
    } else {
      favoriteMealsCtx.addFavorite(mealId);
    }
  }

  useLayoutEffect(() => {
    navigation.setOptions({
      headerRight: () => {
        return (
          <IconButton 
            icon={ mealIsFavorite ? "star": "star-outline"}
            color={ mealIsFavorite ? "yellow": "white"}
            ...
            onPress={changeFavoriteStatusHandler}
          />
        );
      },
    });
  }, [navigation, changeFavoriteStatusHandler]);

  return (
    <ScrollView style={...}>
      <Image style={...} source={{ uri: ... }} />
      ...
    </ScrollView>
  );
}

export default MealDetailScreen;
```

#### ../screens/MealFavorites.tsx
```ts
...
import { FavoritesContext } from '../store/context/favorites-context';
...
function FavoritesScreen() {
  const favoriteMealsCtx = useContext(FavoritesContext);

  const favoriteMeals = MEALS.filter((meal) => 
    favoriteMealsCtx.ids.includes(meal.id)
  );

  if (favoriteMeals.length === 0) {
    return (
        <View style={...}>
            <Text>There is no Favorite Meals</Text>
        </View>
    );
  }

  return (
    <MealList items={favoriteMeals} />
  );
}

export default FavoritesScreen;
```


### Redux

A more powerful state management library that is suited for large and complex applications where state management needs to be predictable and consistent. Comes with a structured way to manage state using actions, reducers, and a central store. Ideal for complex state management needs with large applications where you require a more structured approach, support for middleware, and sophisticated tools for debugging and side effects (Ex: Managing the state of an e-commerce application where you have complex cart operations, user interactions, and asynchronous API calls.).

[Redux Toolkit](https://redux-toolkit.js.org/)

```bash
npm install @reduxjs/toolkit react-redux
```

#### ../store/redux/store.ts
```ts
...
import { configureStore } from '@reduxjs/toolkit';
import favoritesReducer from './favorites';

export const store = configureStore({
    reducer: {}
    favoriteMeals: favoritesReducer
});

```

#### ../App.tsx
```ts
...
import { Provider } from 'react-redux';
import { store } from './store/redux/store';
...
export default function App() {
  return (
    <>
      <StatusBar style="light" />
      <Provider store={store}>
        <NavigationContainer>
          <Stack.Navigator
            screenOptions={{
              ...
            }}
          >
            <Stack.Screen
              name="Drawer"
              component={DrawerNavigator}
              options={{
                headerShown: false,
                ...
              }}
            />
            ...
          </Stack.Navigator>
        </NavigationContainer>
      </Provider>
    </>
  );
}
...
```

#### ../store/redux/favorites.ts
```ts
import { createSlice } from '@reduxjs/toolkit';

const favoritesSlice = createSlice({
    name: 'favorites',
    initialState: {
        ids: []
    },
    reducers: {
        addFavorite: (state, action) => {
            state.ids.push(action.payload.id);  // We use the payload property to transport any of extra data
        },
        removeFavorite: (state) => {
            state.ids.splice(state.ids.indexOf(action.payload.id), 1);
        }
    }
});

export const addFavorite = favoritesSlice.actions.addFavorite;
export const removeFavorite = favoritesSlice.actions.removeFavorite;

export default favoritesSlice.reducer;

```

#### ../screens/MealDetails.tsx
```ts
...
import { useDispatch, useSelector } from 'react-redux';
import { addFavorite, removeFavorite } from '../store/redux/favorites';
...
function MealDetailScreen({ route, navigation }) {
  const favoriteMealIds = useSelector((state) => state.favoriteMeals.ids)
  const dispatch = useDispatch();

  const mealId = route.params.mealId;
  const selectedMeal = MEALS.find((meal) => meal.id === mealId);

  const mealIsFavorite = favoriteMealIds.ids.includes(mealId);

  function changeFavoriteStatusHandler() {
    if (mealIsFavorite) {
      dispatch(removeFavorite({ id: mealId }));
    } else {
      dispatch(addFavorite({ id: mealId }));
    }
  }

  useLayoutEffect(() => {
    navigation.setOptions({
      headerRight: () => {
        return (
          <IconButton 
            icon={ mealIsFavorite ? "star": "star-outline"}
            color={ mealIsFavorite ? "yellow": "white"}
            ...
            onPress={changeFavoriteStatusHandler}
          />
        );
      },
    });
  }, [navigation, changeFavoriteStatusHandler]);

  return (
    <ScrollView style={...}>
      <Image style={...} source={{ uri: ... }} />
      ...
    </ScrollView>
  );
}

export default MealDetailScreen;
```

#### ../screens/MealFavorites.tsx
```ts
...
import { useSelector } from 'react-redux';
...
function FavoritesScreen() {
  const favoriteMealIds = useSelector((state) => state.favoriteMeals.ids)

  const favoriteMeals = MEALS.filter((meal) => 
    favoriteMealIds.includes(meal.id)
  );

  if (favoriteMeals.length === 0) {
    return (
        <View style={...}>
            <Text>There is no Favorite Meals</Text>
        </View>
    );
  }

  return (
    <MealList items={favoriteMeals} />
  );
}

export default FavoritesScreen;
```
