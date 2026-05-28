# 示例代码模板

## 用途

为开发类子智能体提供代码风格参考，确保生成代码的一致性。

---

## 前端 Vue 3 + TypeScript 模板

### 组件模板

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'
import { useXxxStore } from '@/stores/xxx'

interface Props {
  title: string
  count?: number
}

const props = withDefaults(defineProps<Props>(), {
  count: 0
})

const emit = defineEmits<{
  update: [value: string]
}>()

const store = useXxxStore()
const loading = ref(false)

const doubleCount = computed(() => props.count * 2)

const handleClick = () => {
  emit('update', props.title)
}
</script>

<template>
  <div class="xxx-component">
    <h2>{{ title }}</h2>
    <p>Count: {{ doubleCount }}</p>
    <button @click="handleClick" :disabled="loading">
      {{ loading ? 'Loading...' : 'Click' }}
    </button>
  </div>
</template>

<style scoped>
.xxx-component {
  padding: 16px;
}
</style>
```

### Store 模板 (Pinia)

```typescript
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import { fetchXxxList } from '@/api/xxx'
import type { XxxItem } from '@/types/xxx'

export const useXxxStore = defineStore('xxx', () => {
  const items = ref<XxxItem[]>([])
  const loading = ref(false)
  const error = ref<string | null>(null)

  const itemCount = computed(() => items.value.length)

  async function loadItems() {
    loading.value = true
    error.value = null
    try {
      items.value = await fetchXxxList()
    } catch (e) {
      error.value = e instanceof Error ? e.message : 'Unknown error'
    } finally {
      loading.value = false
    }
  }

  return { items, loading, error, itemCount, loadItems }
})
```

---

## 后端 Spring Boot 模板

### Controller 模板

```java
@RestController
@RequestMapping("/api/v1/xxx")
@RequiredArgsConstructor
@Slf4j
public class XxxController {

    private final XxxService xxxService;

    @GetMapping
    public ResponseEntity<PageResponse<XxxVO>> list(
            @RequestParam(defaultValue = "1") int page,
            @RequestParam(defaultValue = "10") int size) {
        return ResponseEntity.ok(xxxService.list(page, size));
    }

    @GetMapping("/{id}")
    public ResponseEntity<XxxVO> get(@PathVariable Long id) {
        return ResponseEntity.ok(xxxService.get(id));
    }

    @PostMapping
    public ResponseEntity<XxxVO> create(@Valid @RequestBody XxxCreateDTO dto) {
        XxxVO vo = xxxService.create(dto);
        return ResponseEntity.status(HttpStatus.CREATED).body(vo);
    }

    @PutMapping("/{id}")
    public ResponseEntity<XxxVO> update(
            @PathVariable Long id,
            @Valid @RequestBody XxxUpdateDTO dto) {
        return ResponseEntity.ok(xxxService.update(id, dto));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        xxxService.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

### Service 模板

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class XxxServiceImpl implements XxxService {

    private final XxxRepository xxxRepository;

    @Override
    public PageResponse<XxxVO> list(int page, int size) {
        Pageable pageable = PageRequest.of(page - 1, size);
        Page<Xxx> xxxPage = xxxRepository.findAll(pageable);
        return PageResponse.of(xxxPage, this::toVO);
    }

    @Override
    public XxxVO get(Long id) {
        Xxx xxx = xxxRepository.findById(id)
            .orElseThrow(() -> new BusinessException(ErrorCode.XXX_NOT_FOUND));
        return toVO(xxx);
    }

    @Override
    @Transactional
    public XxxVO create(XxxCreateDTO dto) {
        Xxx xxx = new Xxx();
        // TODO: map fields from dto
        xxxRepository.save(xxx);
        log.info("Created xxx: {}", xxx.getId());
        return toVO(xxx);
    }

    private XxxVO toVO(Xxx xxx) {
        XxxVO vo = new XxxVO();
        // TODO: map fields
        return vo;
    }
}
```

---

## 区块链 Solidity 模板

### 合约模板

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/access/AccessControl.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

/**
 * @title XxxContract
 * @dev xxx 功能描述
 */
contract XxxContract is AccessControl, ReentrancyGuard {
    bytes32 public constant ADMIN_ROLE = keccak256("ADMIN_ROLE");

    struct XxxItem {
        uint256 id;
        address owner;
        uint256 value;
        bool active;
    }

    mapping(uint256 => XxxItem) private items;
    uint256 private nextId;

    event XxxCreated(uint256 indexed id, address indexed owner, uint256 value);
    event XxxUpdated(uint256 indexed id, uint256 value);

    constructor() {
        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
        _grantRole(ADMIN_ROLE, msg.sender);
    }

    /**
     * @dev 创建 xxx
     * @param value 初始值
     */
    function create(uint256 value) external nonReentrant returns (uint256) {
        require(value > 0, "Value must be positive");

        uint256 id = nextId++;
        items[id] = XxxItem(id, msg.sender, value, true);

        emit XxxCreated(id, msg.sender, value);
        return id;
    }

    /**
     * @dev 更新 xxx
     * @param id xxx ID
     * @param value 新值
     */
    function update(uint256 id, uint256 value) external nonReentrant {
        XxxItem storage item = items[id];
        require(item.active, "Item not found");
        require(item.owner == msg.sender || hasRole(ADMIN_ROLE, msg.sender), "Not authorized");

        item.value = value;
        emit XxxUpdated(id, value);
    }

    /**
     * @dev 获取 xxx
     * @param id xxx ID
     */
    function get(uint256 id) external view returns (uint256, address, uint256, bool) {
        XxxItem storage item = items[id];
        require(item.active, "Item not found");
        return (item.id, item.owner, item.value, item.active);
    }
}
```

---

## Flutter 模板

### Widget 模板

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:xxx/providers/xxx_provider.dart';

class XxxWidget extends ConsumerWidget {
  const XxxWidget({
    super.key,
    required this.title,
    this.count = 0,
  });

  final String title;
  final int count;

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final xxxState = ref.watch(xxxProvider);

    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              title,
              style: Theme.of(context).textTheme.titleLarge,
            ),
            const SizedBox(height: 8),
            Text('Count: $count'),
            const SizedBox(height: 16),
            xxxState.when(
              data: (data) => Text('Data: $data'),
              loading: () => const CircularProgressIndicator(),
              error: (error, stack) => Text('Error: $error'),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Provider 模板

```dart
import 'package:riverpod_annotation/riverpod_annotation.dart';
import 'package:xxx/services/xxx_service.dart';

part 'xxx_provider.g.dart';

@riverpod
class Xxx extends _$Xxx {
  @override
  Future<List<XxxItem>> build() async {
    final service = ref.read(xxxServiceProvider);
    return service.fetchAll();
  }

  Future<void> create(XxxCreateDto dto) async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      final service = ref.read(xxxServiceProvider);
      await service.create(dto);
      return service.fetchAll();
    });
  }

  Future<void> update(String id, XxxUpdateDto dto) async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      final service = ref.read(xxxServiceProvider);
      await service.update(id, dto);
      return service.fetchAll();
    });
  }

  Future<void> delete(String id) async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      final service = ref.read(xxxServiceProvider);
      await service.delete(id);
      return service.fetchAll();
    });
  }
}
```

---

## 前后端联调 API 调用模板

### 前端 API 调用

```typescript
import { request } from '@/utils/request'
import type { ApiResponse, PageResponse } from '@/types/api'
import type { XxxItem, XxxCreateDto, XxxUpdateDto } from '@/types/xxx'

export const xxxApi = {
  /** 获取列表 */
  list(params: { page?: number; size?: number }) {
    return request.get<PageResponse<XxxItem>>('/api/v1/xxx', { params })
  },

  /** 获取详情 */
  get(id: string) {
    return request.get<ApiResponse<XxxItem>>(`/api/v1/xxx/${id}`)
  },

  /** 创建 */
  create(data: XxxCreateDto) {
    return request.post<ApiResponse<XxxItem>>('/api/v1/xxx', data)
  },

  /** 更新 */
  update(id: string, data: XxxUpdateDto) {
    return request.put<ApiResponse<XxxItem>>(`/api/v1/xxx/${id}`, data)
  },

  /** 删除 */
  delete(id: string) {
    return request.delete(`/api/v1/xxx/${id}`)
  },
}
```
