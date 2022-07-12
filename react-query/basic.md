# react-query
- 서버의 값을 클라이언트에 가져오거나 캐싱, 값 업데이트 에러핸들링 등 비동기 과정을 더욱 편하게 하는데 사용.

## why?
- store을 사용하여 개발하다보면 클라이언트 데이터와 서버 데이터가 store에 공존하면서 서로 섞이게 된다. react-query를 통해 이를 분리하여 사용할 수 있다.

## 장점
- 캐싱
- get 데이터를 update하면 자동으로 get이 다시 refetch된다.
- 데이터가 오래되었다고 판단하면 다시 get 요청을 수행한다.
- 동일한 데이터를 중복하여 요청하면 한번만 수행하며 옵션에 따라 중복 호출 허용 시간 조절이 가능하다.
- 비동기 과정을 선언적으로 관리할 수 있다.
- react hook과 사용 구조가 비슷하다.

## 사용
> next.js + typescript 
```tsx
// _app.tsx
...
import { QueryClient, QueryClientProvider } from "react-query";
import { ReactQueryDevtools } from "react-query/devtools";

const queryClient = new QueryClient();

<QueryClientProvider client={queryClient}>
  <Hydrate state={pageProps.dehydratedState}>
      {/* @ts-ignore */}
      <Component {...pageProps} />
  </Hydrate>
  <ReactQueryDevtools />
</QueryClientProvider>
```

### useQuery
- get 요청 시 사용하는 api.
- useQuery(uniqueKey, api호출함수)
- unique Key(string, array)는 다른 컴포넌트에서도 호출이 가능하다.
  + unique Key가 배열인 경우 [unique Key, query 함수 내부로 전달할 파라미터 값]
- 비동기로 작동하기 때문에 한 컴포넌트에 여러개의 useQuery가 있다면 동시에 실행되기 때문에 여러개의 비동기 query가 있다면 useQueries를 사용하는게 좋다.
- enabled를 사용하여 동기적으로 사용이 가능하다.
```jsx
// api.js
export const fetchTodoList = async () => {
  const { data }: AxiosResponse = await instance.get(`${basePath}/getTodo`)
  return data
}

// todo.js
const Todos = () => {
  const { isLoading, isError, data, error } = useQuery("todos", fetchTodoList, {
    refetchOnWindowFocus: false, // 재실행 여부 옵션
    retry: 0,
    onSuccess: data => {
      // 성공시 호출
      console.log(data);
    },
    onError: e => {
      // 실패시 호출
      console.log(e.message);
    }
  });

  if (isLoading) {
    return <span>Loading...</span>;
  }

  if (isError) {
    return <span>Error: {error.message}</span>;
  }

  return (
    <ul>
      {data.map(todo => (
        <li key={todo.id}>{todo.title}</li>
      ))}
    </ul>
  );
};
```
- status로 한번에 처리가 가능하다.
```jsx
const { status } = useQuery("todos", fetchTodoList);

if (status === "loading") {
    return <span>Loading...</span>;
  }

if (status === "error") {
  return <span>Error: {error.message}</span>;
}
```

#### useQuery 동기적 실행.
- enabled가 true면 동기적으로 실행할 수 있다.
```jsx
const [active, setActive] = useState(false)

const { isLoading, isError, data, error } = useQuery("todos", fetchTodoList, {
    enabled: !!active // true가 되면 실행한다.
    refetchOnWindowFocus: false, // 재실행 여부 옵션
    retry: 0,
    onSuccess: data => {
      // 성공시 호출
      console.log(data);
    },
    onError: e => {
      // 실패시 호출
      console.log(e.message);
    }
  });
```

### useQueries
- 여러개의 useQuery를 묶어서 사용한다.
```jsx
const result = useQueries([
  {
    queryKey: "getRune",
    queryFn: () => api.getRunInfo(riot.version)
  },
  {
    queryKey: "getSpell",
    queryFn: () => api.getSpellInfo(riot.version)
  }
]);
```

### unique key의 활용
- unique key가 배열이면 query함수 내부에서 변수로 사용이 가능하다.
```jsx
const riot = {
  version: "12.1.1"
};

const result = useQueries([
  {
    queryKey: ["getRune", riot.version],
    queryFn: params => {
      // unique key를 변수로 확인이 가능하며 이를 이용하여 파라미터로 이용이 가능하다.
      console.log(params); // {queryKey: ['getRune', '12.1.1'], pageParam: undefined, meta: undefined}
      return api.getRunInfo(riot.version);
    }
  },
  {
    queryKey: ["getSpell", riot.version],
    queryFn: () => api.getSpellInfo(riot.version)
  }
]);
```

### useMutation
- get을 제외한 api요청 시 사용.
```jsx
const loginMutation = useMutation(loginApi, {
   // 입력값
    onMutate: variable => {
      console.log(variable);
      // variable : {loginId: 'xxx', password; 'xxx'}
    },
    // 실패시
    onError: (error, variable, context) => {
      // error
    },
    // 성공시
    onSuccess: (data, variables, context) => {
      console.log("success", data, variables, context);
    }
  });

const handleSubmit = () => {
    loginMutation.mutate({ loginId: id, password });
  };

...
<button onClick={handleSubmit}>로그인</button>
```
#### get 다시 실행하기
- 데이터를 업데이트 후 다시 get을 요청할 경우
```jsx
const mutation = useMutation(postTodo, {
  onSuccess: () => {
    // 성공시 get 요청을 하고자 하는 useQuery의 uniueKey를 가져온다.
    queryClient.invalidateQueries(uniueKey);
  }
});
```
- mutation의 return 값을 이용하여 파라미터를 변경해야하는 경우
```jsx
const mutation = useMutation(editTodo, {
  onSuccess: data => {
    // data가 fetchTodoById로 들어간다
    queryClient.setQueryData(["todo", { id: 5 }], data);
  }
});

const { status, data, error } = useQuery(["todo", { id: 5 }], fetchTodoById);
```