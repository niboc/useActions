# useActions
The potential of useReducer by using simple methods
[DEMO](https://codesandbox.io/s/useactions-9uhbz)

## HOOK
```react
import React from "react";

const KEY_UPDATE = "__update";

const useActions = ({ initialState, actions }) => {
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
    decrement(e) {
      update({ count: state.count - 1 });
    },
    increment(e) {
      update({ count: state.count + 1 });
    },
    getRandom(e) {
      update({ loading: true });
      
      //Simulate async call
      tmAux && clearTimeout(tmAux);
      tmAux = setTimeout(() => {
        update({
          loading: false,
          count: Math.ceil(Math.random() * 20)
        });
        tmAux = null;
      }, 3000);
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
  const [{ count, loading }, { decrement, increment, getRandom }] = useActions({
    initialState,
    actions
  });

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
        <button onClick={getRandom}>
          {loading ? "loading..." : "Randomize"}
        </button>
      </div>
    </div>
  );
};

export default Counter;
```

