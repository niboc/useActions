# useActions
The potential of useReducer by using simple methods. When rendering only changes what really changes. Methods retain their original reference. Please leave your comments, they are appreciated.

[DEMO](https://codesandbox.io/s/useactions-9uhbz)

## HOOK
```react
import React from "react";

const KEY_UPDATE = "__update";

const useActions = (actions, initialState = {}) => {
  const keys = React.useMemo(() => {
    return Object.keys(actions());
  }, [actions]);

  const update = newState => {
    dispatch({
      type: KEY_UPDATE,
      payload: {
        ...state,
        ...newState
      }
    });
  };

  const reducer = (state, action) => {
    const _actions = actions(state, update);

    if (action.type === KEY_UPDATE) return action.payload;
    else if (_actions[action.type]) {
      setTimeout(() => {
        _actions[action.type](...action.args);
      }, 1);
      return state;
    } else return state;
  };

  const [state, dispatch] = React.useReducer(reducer, initialState);

  const methods = React.useMemo(() => {
    const aux = {};
    keys.forEach(key => {
      aux[key] = (...args) => {
        dispatch({ args, type: key });
      };
    });
    return aux;
  }, [keys]);

  return [state, methods];
};

export default useActions;
```


## EXAMPLE

###### actions.js
```react
export const initialState = { count: 0, count2: 0, loading: false };

export const actions = (state, update) => {
  let tmAux = null;

  return {
    decrement() {
      update({ count: state.count - 1 });
    },
    increment() {
      update({ count: state.count + 1 });
    },
    getRandom(min, max) {
      //Cancelo request anterior
      tmAux && clearTimeout(tmAux);
      //Simulo delay
      tmAux = setTimeout(() => {
        update({
          loading: false,
          count: Math.ceil(Math.random() * (max - min - 1)) + min - 1
        });
        tmAux = null;
      }, 3000);

      update({ loading: true });
    }
  };
};
```

###### component
```react
import React from "react";
import useActions from "../../hooks/useActions";
import { actions, initialState } from "./actions.js";

const Counter = ({ id }) => {
  const [{ count, loading }, { decrement, increment, getRandom }] = useActions(actions, initialState);

  return (
    <div style={{ marginBottom: 20 }}>
      <div>
        <h3>Counter {id}</h3>
        <button onClick={decrement}>-</button>
        <input
          type="text"
          onChange={() => {}}
          value={count}
          style={{ textAlign: "center" }}
        />
        <button onClick={increment}>+</button>
      </div>
      <div>
        <button onClick={() => getRandom(20, 30)}>
          {loading ? "loading..." : "Randomize"}
        </button>
      </div>
    </div>
  );
};

export default Counter;
```

