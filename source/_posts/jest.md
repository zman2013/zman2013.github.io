---
title: jest
date: 2021-05-22 09:39:17
tags:
---

## mock function，判断被调用次数、参数
```typescript
  const abort = jest.fn()
  ...
  expect((abort as jest.Mock).mock.calls.length).toBe(1)
  // 第一次调用的第一个参数
  expect((abort as jest.Mock).mock.calls[0][0]).toBe(arg1)
```

## mock function, 设置返回数据
```typescript
  const connect = jest.fn()
  (connect as jest.Mock).mockReturnValue('success')
```

## mock object
```typescript
  const aObject = {} as any
```

## mock 3p module
```typescript
  import { axios } from 'axios'
  jest.mock('axios')

  describe('basic', ()=>{
    beforeEach(()=>{
      // 执行每个单元测试前，清除 getLog 调用信息
      (axios.request as jest.Mock).mockClear()
    })

    it('case1', ()=>{
      (axios.request as jest.Mock).mockReturnValue({...})
    })
  })
```

## mock local function & class 
```typescript
  import { IntClusterRebalancer } from '../src/int-cluster-rebalancer'
  let fn1
  jest.mock('../src/int-cluster-rebalancer', () => {
    fn1 = jest.fn()
    return {
      functionName: fn1,
      IntClusterRebalancer: jest.fn().mockImplementation(() => {
        return { rebalance: jest.fn() }
      }),
    }
  })

  describe('basic', () => {

    beforeEach(() => {
      ;(IntClusterRebalancer as jest.Mock).mockClear()
    })

    it('case1', ()=>{
      const rebalancer = new IntClusterRebalancer()
      // or 任何内部执行 new IntClusterRebalancer() 的方法/对象
      const manager = new Manager() // Manager 内部会执行 new IntClusterRebalancer()
    })
  })
```