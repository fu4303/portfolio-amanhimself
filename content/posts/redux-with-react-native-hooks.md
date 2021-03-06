---
title: 'Using Redux with React Hooks in a React Native app'
date: 2020-01-27
slug: 'blog/redux-with-react-native-hooks'
thumbnail: '../thumbnails/expo.png'
template: post
tags:
  - expo
  - redux
  - react-native
---

With React Hooks growing usage, the ability to handle a component's state and side effects is now a common pattern in the functional component. React Redux offers a set of Hook APIs as an alternative to the omnipresent `connect()` High Order Component.

In this tutorial, let us continue to build a simple React Native app where a user can save their notes and let use Redux Hooks API to manage state. This post is in continuation of the previous post [here](https://heartbeat.fritz.ai/getting-started-with-react-native-and-expo-using-hooks-in-2020-fb466c25b04c).

If you are familiar with the basics of React Hooks and how to implement them with a basic navigation setup, you can skip the previous post and can continue from this one.

## Table of Contents

- Installing redux
- Adding action types and creators
- Add a reducer
- Configuring a redux store
- Accessing global state
- Dispatching actions
- Running the app
- Conclusion

## Installing redux

If you have cloned the repo from the previous example, make sure that the `dependencies` in the `package.json` file looks like below:

```json
"dependencies": {
 "@react-native-community/masked-view": "0.1.5",
 "expo": "~36.0.0",
 "react": "~16.9.0",
 "react-dom": "~16.9.0",
 "react-native": "https://github.com/expo/react-native/archive/sdk-36.0.0.tar.gz",
 "react-native-gesture-handler": "~1.5.0",
 "react-native-paper": "3.4.0",
 "react-native-reanimated": "~1.4.0",
 "react-native-safe-area-context": "0.6.0",
 "react-native-screens": "2.0.0-alpha.12",
 "react-navigation": "4.0.10",
 "react-navigation-stack": "2.0.10",
 "react-redux": "7.1.3",
 "redux": "4.0.5"
 },
```

Next, install the following dependencies from a terminal window to integrate and use Redux to manage the state.

```shell
yarn add redux react-redux lodash.remove
```

The directory structure that I am going to follow to manage Redux related files is going to be based on the pragmatic approach called ducks. [Here is the link](https://medium.com/swlh/the-good-the-bad-of-react-redux-and-why-ducks-might-be-the-solution-1567d5bdc698) to a great post on using ducks pattern in Redux and React apps. This post can help you understand the pattern and why there can be a requirement for it.

What ducks pattern allows you to have are modular reducers in the app itself. You do not have to create different files for actions, types, and action creators. Instead, you can define them all in one modular file however, if there is a need to create more than one reducer, you can have defined multiple reducer files.

## Adding action types and creators

When using Redux to manage the state of the whole application, the state itself is represented by one JavaScript object. Think of this object as read-only, since you cannot make changes to this state (_which is represented in the form of a tree_) directly. It requires actions to do so.

Actions are like events in Redux. They can be triggered in the button press, timers or network requests.

To begin, inside the `src/` directory, create a subdirectory called `redux`. Inside it, create a new file called `notesApp.js`.

So far, the application has the ability to let the user add notes. In the newly created files, let us begin by defining two action types and their creators. The second action type is going to allow the user to remove an item from the `ViewNotes` screen.

```js
// Action Types

export const ADD_NOTE = 'ADD_NOTE';
export const DELETE_NOTE = 'DELETE_NOTE';
```

Next, let us define action creators for each of the action type. The first one is going trigger when saving the note. The second creator is going to trigger when deleting the note.

```js
// Action Creators

let noteID = 0;

export function addnote(note) {
  return {
    type: ADD_NOTE,
    id: noteID++,
    note
  };
}

export function deletenote(id) {
  return {
    type: DELETE_NOTE,
    payload: id
  };
}
```

## Add a reducer

The receiver of the action is known as a reducer. Whenever an action is triggered, the state of the application changes. The handling of the application???s state is done by the reducers.

A reducer is a pure function that calculates the next state based on the initial or previous state. It always produces the same output if the state is unchanged. It takes two inputs, the `state` and `action` and must return the default `state`.

The initial state is going to be an empty array. Add the following after you have defined action creators. Also, make sure to import `remove` utility from `lodash.remove` npm package at the top of the file `notesApp.js` that was installed at the starting of this post.

```js
// import the dependency
import remove from 'lodash.remove';

// reducer

const initialState = [];

function notesReducer(state = initialState, action) {
  switch (action.type) {
    case ADD_NOTE:
      return [
        ...state,
        {
          id: action.id,
          note: action.note
        }
      ];

    case DELETE_NOTE:
      const deletedNewArray = remove(state, obj => {
        return obj.id != action.payload;
      });
      return deletedNewArray;

    default:
      return state;
  }
}

export default notesReducer;
```

## Configuring a redux store

A store is an object that brings and actions and reducers together. It provides and holds state at the application level instead of individual components. Redux is not an opinionated library in terms of which framework or library should use it or not.

With the creation of reducer done, create a new file called `store.js` inside `src/redux/`. Import the function `createStore` from `redux` as well as the only reducer in the app for now.

```js
import { createStore } from 'redux';
import notesReducer from './notesApp';

const store = createStore(notesReducer);

export default store;
```

To bind this Redux store in the React Native app, open the entry point file `App.js` and import the `store` as well as the High Order Component `Provider` from `react-redux` npm package. This HOC helps you to pass the store down to the rest of the components of the current app.

```js
import React from 'react';
import { Provider as PaperProvider } from 'react-native-paper';
import AppNavigator from './src/navigation';
import { Provider as StoreProvider } from 'react-redux';
import store from './src/redux/store';

// modify the App component
export default function App() {
  return (
    <StoreProvider store={store}>
      <PaperProvider>
        <AppNavigator />
      </PaperProvider>
    </StoreProvider>
  );
}
```

That's it! The Redux store is now configured and ready to use.

## Accessing global state

To access state when managing it with Redux, `useSelector` hook is provided. It is similar to `mapStateToProps` argument that is passed inside the `connect()`. It allows you to extract data from the Redux store state using a selector function.

The major difference between the hook and the argument is that the selector may return any value as a result, not just an object.

Open `ViewNotes.js` file and import this hook from `react-redux`.

```js
// ...after rest of the imports
import { useSelector } from 'react-redux';
```

Next, instead of storing `notes` array using `useState` Hook, replace it with the following inside `ViewNotes` functional component.

```js
const notes = useSelector(state => state);
```

## Dispatching actions

The `useDispatch()` hook completely refers to the dispatch function from the Redux store. This hook is used only when there is a need to dispatch an action. Import it from `react-redux` and also, the action creators `addnote` and `deletenote` from the file `redux/notesApp.js`.

```js
import { useSelector, useDispatch } from 'react-redux';
```

To dispatch an action, define the following statement after the `useSelector` hook.

```js
const dispatch = useDispatch();
```

Next, dispatch two actions called `addNote` and `deleteNote` to trigger these events.

```js
const addNote = note => dispatch(addnote(note));
const deleteNote = id => dispatch(deletenote(id));
```

Since the naming convention is exactly the same for the `addNote` action as from the previous post's helper function, there is no need to make any changes inside the `return` statement of the functional component for this. However, `deleteNote` action is new.

To delete a note from the list rendered, add a prop `onPress` to `List.Item` UI component from `react-native-paper`. This is going to add the functionality of deleting an item from the list when the user touches that item.

Here is the code snippet of `List.Item` component with changes. Also, make sure that to modify the values of props: `title` and `description`.

```js
<List.Item
  title={item.note.noteTitle}
  description={item.note.noteValue}
  descriptionNumberOfLines={1}
  titleStyle={styles.listTitle}
  onPress={() => deleteNote(item.id)}
/>
```

The advantage `useDispatch` hook provides is that it replaces `mapDispatchToProps` and there is no need to write boilerplate code to bind action creators with this hook now.

## Running the app

So far so good. Now, let us run the application. From the terminal window execute the command `expo start` or `yarn start` and make sure the Expo client is running on a simulator or a real device. You are going to be welcomed by the following home screen that currently has no notes to display.

![hb1](https://miro.medium.com/max/375/1*xE1d1MBk-uJrU8UlD-KW6A.png)

Here is the complete demo that showcases both adding a note and deleting a note functionality.

![hb2](https://miro.medium.com/max/381/1*BJYPQHfTF0wKbZ9CIIhL0g.gif)

For you reference, there are no changes made inside the `AddNotes.js` file and it still uses the `useState` to manage the component's state. There are quite a few changes made to `ViewNotes.js` file so here is the complete snippet of code:

```js
// ViewNotes.js
import React from 'react';
import { StyleSheet, View, FlatList } from 'react-native';
import { Text, FAB, List } from 'react-native-paper';
import { useSelector, useDispatch } from 'react-redux';
import { addnote, deletenote } from '../redux/notesApp';

import Header from '../components/Header';

function ViewNotes({ navigation }) {
  // const [notes, setNotes] = useState([])

  // const addNote = note => {
  // note.id = notes.length + 1
  // setNotes([...notes, note])
  // }

  const notes = useSelector(state => state);
  const dispatch = useDispatch();
  const addNote = note => dispatch(addnote(note));
  const deleteNote = id => dispatch(deletenote(id));

  return (
    <>
      <Header titleText="Simple Note Taker" />
      <View style={styles.container}>
        {notes.length === 0 ? (
          <View style={styles.titleContainer}>
            <Text style={styles.title}>You do not have any notes</Text>
          </View>
        ) : (
          <FlatList
            data={notes}
            renderItem={({ item }) => (
              <List.Item
                title={item.note.noteTitle}
                description={item.note.noteValue}
                descriptionNumberOfLines={1}
                titleStyle={styles.listTitle}
                onPress={() => deleteNote(item.id)}
              />
            )}
            keyExtractor={item => item.id.toString()}
          />
        )}
        <FAB
          style={styles.fab}
          small
          icon="plus"
          label="Add new note"
          onPress={() =>
            navigation.navigate('AddNotes', {
              addNote
            })
          }
        />
      </View>
    </>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    paddingHorizontal: 10,
    paddingVertical: 20
  },
  titleContainer: {
    alignItems: 'center',
    justifyContent: 'center',
    flex: 1
  },
  title: {
    fontSize: 20
  },
  fab: {
    position: 'absolute',
    margin: 20,
    right: 0,
    bottom: 10
  },
  listTitle: {
    fontSize: 20
  }
});

export default ViewNotes;
```

## Conclusion

With the addition to hooks such as `useSelector` and `useDispatch` not only reduces the need to write plentiful boilerplate code but also gives you the advantage to use functional components.

For advanced usage of Hooks with Redux, you can check out the official documentation [**here**](https://react-redux.js.org/next/api/hooks).

You can find the complete code for this tutorial in the [Github repo here](https://github.com/amandeepmittal/rnReduxhooks).

---

Originally published at [Heartbeat.fritz.ai](https://heartbeat.fritz.ai/using-redux-with-react-hooks-in-a-react-native-app-cc410a77f3e2)
