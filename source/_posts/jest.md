---
title: jest
date: 2021-05-22 09:39:17
tags:
---

## mock function，判断被调用次数
```typescript
  const abort = jest.fn()
  ...
  expect((abort as jest.Mock).mock.calls.length).toBe(1)
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
  import { getLog } from 'nestjs-log'
  jest.mock('nestjs-log')

  describe('basic', ()=>{
    beforeEach(()=>{
      // 执行每个单元测试前，清除 getLog 调用信息
      (getLog as jest.Mock).mockClear()
    })

    it('case1', ()=>{
      (connect as jest.Mock).mockReturnValue({...})
    })
  })
```

## mock local class
```typescript
  import { IntClusterRebalancer } from '../src/int-cluster-rebalancer'
  jest.mock('../src/int-cluster-rebalancer', () => {
    return {
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