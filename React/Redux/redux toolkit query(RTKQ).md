> RTK Query是一个强大的数据获取和缓存工具，在它的帮助下，web应用的加载变得十分简单，它使我们不再需要自己编写数据获取和缓存数据的逻辑。

## 创建Api切片

RTKQ中将一组相关功能统一封装到一个api对象中。

`store/studentApi.js`文件

```react
import {createApi, featchBaseQuery} from "@reduxjs/toolkit/dist/query/react";

// createApi()用来创建RTKQ中的API对象
// RTKQ的所有功能都需要通过该对象来进行
const studentApi = createApi({
  reducerPath: 'studentApi', //Api的标识，不能和其他Api或reducer重复
  baseQuery: featchBaseQuery({baseUrl: "http://localhost:1337/api/"}), // 指定查询的基础信息
  // endpoints用来指定Api中的各种功能，是一个方法，需要一个对象作为返回值。
  tagTypes: ['student'], // 用来指定Api中的标签类型
  endpoints(build){
    // build是请求的构建器，通过build来设置请求的相关信息
    return {
      getStudents: build.query({
        query(){
          // 用来设置请求子路径
          return 'students';
        },
        // transformResponse 用来转换响应数据格式
        transformResponse(baseQueryReturnValue){
          return baseQueryReturnValue.data;
        },
        providesTags: ['student'], // 被调者
      }),
      getStudentById: build.query({
        query(id){
          return `students/${id}`;
        },
        // transformResponse 用来转换响应数据格式
        transformResponse(baseQueryReturnValue){
          return baseQueryReturnValue.data;
        },
        keepUnusedDataFor: 0, // 设置数据缓存的时间，单位(秒)。默认是60秒
        providesTags: (result, error, id) => {
          return [{
          type: 'student',
          id,
        }]
        }
      }),
      delStudent: build.mutation({
        query(id){
          return {
            url: `students/${id}`,
            method: 'delete'
          }
        }
      }),
      updateStudent: build.mutation({
        query(stu){
          return {
            url: `students/${stu.id}`,
            method: 'put',
            body: {
              data: stu.data
            }
          }
        },
        invalidatesTags: (result, error, stu) => {
          return [{ type: 'student', id: stu.id }], // 调用者
        }
      })
    }
  }
})

// Api对象创建后，对象中会根据各种方法自动生成对应的钩子函数
// 通过这些钩子函数，可以向服务器发送请求
// 钩子函数命名规则：getStudents --> useGetStudentsQuery
const { useGetStudentsQuery } = studentApi;
export default studentApi;
```

## 创建store对象

> Api对象的使用有两种方式：一种是直接使用，一种是作为store中的一个reducer使用。

`store/index.js`文件

```js
import { configureStore } from '@reduxjs/toolkit';
import { studentApi } from './studentApi';

export const store = configureStore({
  reducer: {
    [studentApi.reducerPath]: studentApi.reducer
  },
  // 中间件用来处理Api的缓存
  middleware: getDefaultMiddleware => {
    // 把Api中生成的中间件配置进默认中间件中去
    return getDefaultMiddleware().concat(studentApi.middleware);
  };
})
```

## 使用

`App.js`

```react
import { useGetStudentsQuery, useGetStudentByIdQuery, useDelStudentMutation, useUpdateStudentMutation } from 'store/studentApi';
const App = (props) => {
  // 调用Api中的钩子查询数据
  // 这个钩子函数返回一个对象作为返回值，请求过程中的相关数据都存储在该对象中
  const { data, isSuccess, isLoading } = useGetStudentsQuery();
  const { data, isSuccess, isLoading } = useGetStudentByIdQuery(props.id); // 更具id获取数据
  const { data, isSuccess, isLoading } = useGetStudentByIdQuery(props.id, {
    // useQuery 还可以接受一个对象作为第二个参数，通过该对象可以对请求进行配置。
    selectFromResult: result => result, // 用来指定useQuery的返回结果
    pollingInterval: 0, // 设置轮询的间隔，单位ms,为0则不轮询
    skip: !props.id, // 设置是否跳过当前请求，默认 false
    
    // 一、值为boolean, 表示设置是否每一次都重新加载数据。false:正常使用缓存，true:每次重载数据。
    // 二、值为number，表示数据缓存(有效期)的时间，单位(秒)
    refetchOnMountOrArgChange: false, 
    refetchOnFocus: false, // 设置是否在页面重新获取焦点时请求数据
    refetchOnReconnect: true, // 设置是否在网络重连后请求数据
  });
  
  // useMutation钩子返回的是一个数组：[操作的触发器，请求返回的结果集]
  const { delStudent, result } = useDelStudentMutation();
  const { updateStudent, result } = useUpdateStudentMutation()
  const handler = () => {
    delStudent(props.id);
    updateStudent({
      id: xx,
      data: xxx
    })
  }
  return (
  	<div>
      { isLoading && <p>数据加载中...</p> }
      { isSuccess && data.data.map(item => <p key={item.id}>...</p>) }
    </div>
  )
}
export default App;

// 更新后
```

