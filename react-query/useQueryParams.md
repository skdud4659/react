# useQuery > 버튼 클릭 시 특정 값을 파라미터로 api 호출하기.
- 맵을 사용하여 리스트가 그려지고 있고 그 중 특정 열의 랭킹 버튼을 클릭하면 해당 아이템의 id값을 파라미터로 보내 get 요청을 한다.

## components.tsx
```jsx
// 파라미터로 보낼 데이터 상태
const [key, setKey] = useState<string>('')
// useQuery 동기 설정 상태
const [queryActive, setQueryActive] = useState<boolean>(false)

useQuery(
  // 값을 useQuery로 호출 시 파라미터로 보내기 위해 unique key를 배열로 만든다.
  ['slotRank', { searchKey: key }],
  // console.log(params) > queryKey: {'slotRank', 'key'} .... 이런 식으로 나온다.
  (params) => fetchSlotRankApi(params.queryKey),
  {
    // useQuery를 동기로 사용하게 설정해준다.
    enabled: queryActive,
    onSuccess: data => {
      setRank(data)
    }
  }
)
...
// 맵 안의 버튼에서 호출
const handleClickBtn = (id: string) => {
  setKey(id)
}
```

## api.ts
```js
export const fetchSlotRankApi = async (params: any) => {
  const { searchKey } = params[1]
  const param: TSlotRankingParam = { searchKey }
  const { data: {data} }: AxiosResponse = await instance.get(`${basePath}/rank/getRank${paramToString(param)}`)
  return data
}
```