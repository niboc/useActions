# useActions
The potential of useReducer by using simple methods

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
        _actions[action.type](action);
      }, 1);
      return state;
    } else return state;
  };

  const [state, dispatch] = React.useReducer(reducer, initialState);

  const methods = React.useMemo(() => {
    const aux = {};
    keys.forEach(key => {
      aux[key] = args => {
        dispatch({ ...args, type: key });
      };
    });
    return aux;
  }, [keys]);

  return [state, methods];
};

export default useActions;
```
