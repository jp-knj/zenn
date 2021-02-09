---
title: "Master to Hooks" 
emoji: "💪🏻" 
type: "tech" 
topics: ["react", "typescript"] 
published: false
---
## はじめに
These are the most common hooks and likely 99% of what you're going to use.
## 3つの必須Hooks

### useState

```javascript
import React, { useState } from "react";

const useStateComponent = () => {
  const [isBlack, setIsBlack] = useState(true);

  return (
    <h1
      onClick={() => setIsBlack(!isBlack)}
      style={{ color: is ? "black" : "white" }}
    >
      useStateの使い方
    </h1>
  );
};

export default useStateComponent;
```

### useEffect
```javascript
import React, { useState, useEffect } from "react";

const useEffectComponent = () => {
  const [time, setTime] = useState(new Date());

  useEffect(() => {
    const timer = setTimeout(setTime(new Date()), 1000);
    return () => clearTimeout(timer);
  });

  return <h1>useEffectの使い方: {time.toLocaleTimeString()}</h1>;
};

export default useEffectComponent;

```

useContext

```javascript
import React, { useState, useContext, createContext } from "react";

const UserContext = createContext([
  {
    firstName: "Yamada",
    lastName: "Tarou",
    suffix: 1,
    email: "tarou.yamada@example.com"
  },
  obj => obj
]);

const LevelFive = () => {
  const [userProfile, setUserProfile] = useContext(UserContext);

  return (
    <div>
      <h5>{`${userProfile.firstName} ${userProfile.lastName} the ${userProfile.suffix} born`}</h5>
      <button
        onClick={() => {
          setUser(Object.assign({}, userProfile, { suffix: userProfile.suffix + 1 }));
        }}
      >
         増加する
      </button>
    </div>
  );
};

const LevelFour = () => (
  <div>
    <h4>fourth level</h4>
    <LevelFive />
  </div>
);

const LevelThree = () => (
  <div>
    <h3>third level</h3>
    <LevelFour />
  </div>
);

const LevelTwo = () => (
  <div>
    <h2>second level</h2>
    <LevelThree />
  </div>
);

const ContextComponent = () => {
  const userState = useState({
    firstName: "James",
    lastName: "Jameson",
    suffix: 1,
    email: "jamesjameson@example.com"
  });

  return (
    <UserContext.Provider value={userState}>
      <h1>first level</h1>
      <LevelTwo />
    </UserContext.Provider>
  );
};

export default ContextComponent;
```

## 理解を深める7つのHooks

useRef


useRender


useMemo


useCallback


useLayoutEffect


useImperativeHandle
