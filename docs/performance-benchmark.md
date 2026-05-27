# 性能基准测试

本文档定义了系统的性能基准测试机制，包括基准建立、回归检测、性能优化和监控告警。

## 1. 性能基准建立

### 1.1 性能指标定义

```json
{
  "performanceMetrics": {
    "responseTime": {
      "description": "响应时间",
      "unit": "milliseconds",
      "thresholds": {
        "excellent": 100,
        "good": 300,
        "acceptable": 500,
        "poor": 1000
      }
    },
    "throughput": {
      "description": "吞吐量",
      "unit": "requests_per_second",
      "thresholds": {
        "excellent": 1000,
        "good": 500,
        "acceptable": 100,
        "poor": 50
      }
    },
    "memoryUsage": {
      "description": "内存使用",
      "unit": "megabytes",
      "thresholds": {
        "excellent": 100,
        "good": 200,
        "acceptable": 500,
        "poor": 1000
      }
    },
    "cpuUsage": {
      "description": "CPU使用率",
      "unit": "percent",
      "thresholds": {
        "excellent": 20,
        "good": 40,
        "acceptable": 60,
        "poor": 80
      }
    }
  }
}
```

### 1.2 基准数据收集

```json
{
  "baselineCollection": {
    "enabled": true,
    "frequency": "per_build",
    "storage": {
      "type": "file",
      "path": "{PROJECT_ROOT}/outputs/performance-baseline.json",
      "format": "json"
    },
    "metrics": [
      "response_time",
      "throughput",
      "memory_usage",
      "cpu_usage",
      "disk_io",
      "network_io"
    ],
    "scenarios": [
      {
        "name": "normal_load",
        "description": "正常负载",
        "users": 100,
        "duration": "5m"
      },
      {
        "name": "peak_load",
        "description": "峰值负载",
        "users": 500,
        "duration": "5m"
      },
      {
        "name": "stress_test",
        "description": "压力测试",
        "users": 1000,
        "duration": "10m"
      }
    ]
  }
}
```

### 1.3 基准数据结构

```json
{
  "performanceBaseline": {
    "version": "1.0",
    "timestamp": "yymmdd hhmm",
    "environment": {
      "os": "linux",
      "cpu": "8 cores",
      "memory": "16GB",
      "disk": "SSD 500GB"
    },
    "scenarios": [
      {
        "name": "normal_load",
        "metrics": {
          "response_time": {
            "avg": 150,
            "p50": 120,
            "p95": 300,
            "p99": 500,
            "max": 800
          },
          "throughput": {
            "avg": 500,
            "min": 400,
            "max": 600
          },
          "memory_usage": {
            "avg": 200,
            "min": 150,
            "max": 300
          },
          "cpu_usage": {
            "avg": 30,
            "min": 20,
            "max": 50
          }
        }
      }
    ]
  }
}
```

---

## 2. 性能回归检测

### 2.1 回归检测规则

```json
{
  "regressionDetection": {
    "enabled": true,
    "rules": [
      {
        "name": "response_time_regression",
        "metric": "response_time",
        "threshold": 20,
        "comparison": "percentage_increase",
        "severity": "major",
        "action": "alert"
      },
      {
        "name": "throughput_regression",
        "metric": "throughput",
        "threshold": 15,
        "comparison": "percentage_decrease",
        "severity": "major",
        "action": "alert"
      },
      {
        "name": "memory_regression",
        "metric": "memory_usage",
        "threshold": 25,
        "comparison": "percentage_increase",
        "severity": "minor",
        "action": "warn"
      },
      {
        "name": "cpu_regression",
        "metric": "cpu_usage",
        "threshold": 20,
        "comparison": "percentage_increase",
        "severity": "minor",
        "action": "warn"
      }
    ]
  }
}
```

### 2.2 回归检测流程

```markdown
### 回归检测流程

1. **收集当前性能数据**
   - 运行性能测试
   - 收集性能指标
   - 记录测试环境

2. **加载历史基准数据**
   - 读取基准文件
   - 验证数据完整性
   - 选择比较基准

3. **比较分析**
   - 计算指标变化
   - 检测回归阈值
   - 识别回归原因

4. **生成报告**
   - 生成回归报告
   - 标记回归指标
   - 提供优化建议

5. **执行动作**
   - 告警通知
   - 阻断构建（可选）
   - 记录历史
```

### 2.3 回归检测报告

```json
{
  "regressionReport": {
    "timestamp": "yymmdd hhmm",
    "buildId": "build-123",
    "summary": {
      "totalMetrics": 10,
      "regressions": 2,
      "improvements": 1,
      "stable": 7
    },
    "regressions": [
      {
        "metric": "response_time",
        "baseline": 150,
        "current": 200,
        "change": 33.3,
        "threshold": 20,
        "severity": "major",
        "suggestion": "检查数据库查询优化"
      }
    ],
    "improvements": [
      {
        "metric": "throughput",
        "baseline": 500,
        "current": 550,
        "change": 10,
        "threshold": 15,
        "severity": "positive"
      }
    ]
  }
}
```

---

## 3. 性能优化建议

### 3.1 瓶颈识别

```json
{
  "bottleneckIdentification": {
    "enabled": true,
    "analysis": {
      "database": {
        "indicators": [
          "slow_queries",
          "missing_indexes",
          "n_plus_one_queries",
          "connection_pool_exhaustion"
        ],
        "thresholds": {
          "query_time": 100,
          "connection_count": 50,
          "lock_wait_time": 50
        }
      },
      "application": {
        "indicators": [
          "memory_leaks",
          "cpu_intensive_operations",
          "blocking_io",
          "inefficient_algorithms"
        ],
        "thresholds": {
          "memory_growth_rate": 10,
          "cpu_usage": 80,
          "io_wait": 30
        }
      },
      "network": {
        "indicators": [
          "high_latency",
          "bandwidth_saturation",
          "connection_timeouts",
          "dns_resolution"
        ],
        "thresholds": {
          "latency": 100,
          "packet_loss": 1,
          "connection_time": 50
        }
      }
    }
  }
}
```

### 3.2 优化建议生成

```json
{
  "optimizationSuggestions": {
    "enabled": true,
    "categories": {
      "database": [
        {
          "issue": "slow_queries",
          "suggestion": "优化慢查询，添加索引",
          "priority": "high",
          "impact": "high"
        },
        {
          "issue": "n_plus_one_queries",
          "suggestion": "使用JOIN或批量查询",
          "priority": "high",
          "impact": "high"
        }
      ],
      "application": [
        {
          "issue": "memory_leaks",
          "suggestion": "检查未释放的资源",
          "priority": "high",
          "impact": "high"
        },
        {
          "issue": "cpu_intensive",
          "suggestion": "优化算法或使用缓存",
          "priority": "medium",
          "impact": "medium"
        }
      ],
      "network": [
        {
          "issue": "high_latency",
          "suggestion": "使用CDN或优化网络配置",
          "priority": "medium",
          "impact": "medium"
        }
      ]
    }
  }
}
```

### 3.3 优化跟踪

```json
{
  "optimizationTracking": {
    "enabled": true,
    "tracking": {
      "before": {
        "response_time": 200,
        "throughput": 500,
        "memory_usage": 300
      },
      "after": {
        "response_time": 150,
        "throughput": 600,
        "memory_usage": 250
      },
      "improvement": {
        "response_time": 25,
        "throughput": 20,
        "memory_usage": 16.7
      }
    }
  }
}
```

---

## 4. 性能测试工具

### 4.1 负载测试工具

```json
{
  "loadTestingTools": {
    "artillery": {
      "enabled": true,
      "config": {
        "target": "http://localhost:3000",
        "phases": [
          {"duration": 60, "arrivalRate": 10},
          {"duration": 120, "arrivalRate": 50},
          {"duration": 60, "arrivalRate": 100}
        ]
      }
    },
    "k6": {
      "enabled": false,
      "config": {
        "target": "http://localhost:3000",
        "stages": [
          {"duration": "1m", "target": 100},
          {"duration": "5m", "target": 100},
          {"duration": "1m", "target": 0}
        ]
      }
    },
    "jmeter": {
      "enabled": false,
      "config": {
        "threads": 100,
        "ramp_up": 60,
        "loop_count": 10
      }
    }
  }
}
```

### 4.2 性能监控工具

```json
{
  "performanceMonitoringTools": {
    "prometheus": {
      "enabled": true,
      "config": {
        "scrape_interval": "15s",
        "scrape_timeout": "10s",
        "metrics_path": "/metrics"
      }
    },
    "grafana": {
      "enabled": true,
      "config": {
        "dashboards": ["performance", "system", "application"]
      }
    },
    "newrelic": {
      "enabled": false,
      "config": {
        "app_name": "My Application",
        "license_key": null
      }
    }
  }
}
```

### 4.3 性能分析工具

```json
{
  "profilingTools": {
    "node": {
      "clinic": {
        "enabled": true,
        "tools": ["doctor", "flame", "bubbleprof"]
      },
      "0x": {
        "enabled": false,
        "config": {
          "output": "flamegraph"
        }
      }
    },
    "java": {
      "jprofiler": {
        "enabled": false,
        "config": {
          "port": 8849
        }
      },
      "visualvm": {
        "enabled": false,
        "config": {
          "port": 9090
        }
      }
    }
  }
}
```

---

## 5. 集成到CI/CD

### 5.1 CI/CD集成配置

```json
{
  "cicdIntegration": {
    "stages": {
      "performance_test": {
        "enabled": true,
        "when": "on_merge",
        "commands": [
          "npm run test:performance",
          "npm run test:load"
        ],
        "artifacts": [
          "performance-report.json",
          "load-test-report.html"
        ]
      },
      "regression_check": {
        "enabled": true,
        "when": "on_merge",
        "commands": [
          "npm run performance:regression"
        ],
        "fail_on_regression": true
      }
    }
  }
}
```

### 5.2 性能门禁

```json
{
  "performanceGates": {
    "enabled": true,
    "gates": [
      {
        "name": "response_time_gate",
        "metric": "response_time.p95",
        "threshold": 500,
        "action": "block"
      },
      {
        "name": "throughput_gate",
        "metric": "throughput.avg",
        "threshold": 100,
        "action": "block"
      },
      {
        "name": "memory_gate",
        "metric": "memory_usage.max",
        "threshold": 500,
        "action": "warn"
      }
    ]
  }
}
```

---

## 6. 集成到主代理

### 6.1 主代理性能测试流程

```markdown
### 性能测试流程

1. **基准建立**
   - 运行性能测试
   - 收集性能指标
   - 保存基准数据

2. **回归检测**
   - 比较当前与基准
   - 检测性能回归
   - 生成回归报告

3. **瓶颈识别**
   - 分析性能瓶颈
   - 识别优化点
   - 生成优化建议

4. **优化跟踪**
   - 记录优化前数据
   - 实施优化措施
   - 记录优化后数据
   - 计算改进效果

5. **报告生成**
   - 生成性能报告
   - 标记问题区域
   - 提供优化建议
```

### 6.2 主代理提示词更新

在主代理提示词中添加性能基准测试：

```markdown
#### 性能基准测试

**基准建立**：
- 记录初始性能指标
- 建立性能基准数据库
- 定期更新基准

**回归检测**：
- 每次提交检测性能变化
- 退化超过阈值报警
- 生成性能趋势图

**性能优化**：
- 自动识别性能瓶颈
- 提供优化建议
- 跟踪优化效果

**监控告警**：
- 实时监控性能指标
- 异常时触发告警
- 生成告警报告
```

---

## 7. 最佳实践

### 7.1 基准建立

1. **代表环境**：在代表性环境建立基准
2. **稳定状态**：系统稳定时收集基准
3. **多次采样**：多次采样取平均值
4. **记录环境**：记录测试环境信息
5. **定期更新**：定期更新基准数据

### 7.2 回归检测

1. **合理阈值**：设置合理的回归阈值
2. **及时检测**：每次提交都检测
3. **阻断严重回归**：严重回归阻断构建
4. **分析原因**：回归时分析根本原因
5. **快速修复**：发现回归快速修复

### 7.3 性能优化

1. **测量优先**：优化前先测量
2. **瓶颈导向**：针对瓶颈优化
3. **逐步优化**：逐步实施优化
4. **验证效果**：优化后验证效果
5. **持续监控**：持续监控性能

### 7.4 监控告警

1. **关键指标**：监控关键性能指标
2. **合理阈值**：设置合理告警阈值
3. **及时响应**：告警时及时响应
4. **分析根因**：分析告警根本原因
5. **持续改进**：持续改进监控策略
