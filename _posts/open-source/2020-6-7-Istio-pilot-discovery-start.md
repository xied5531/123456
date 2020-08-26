---
title: Istio：pilot-discovery启停控制
date: 2020-6-7 8:40:52
tags: [Istio, Pilot]
---

pilot/cmd/pilot-discovery/main.go

## 启动

```go
......
	discoveryCmd = &cobra.Command{
		Use:   "discovery",
		Short: "Start Istio proxy discovery service.",
		Args:  cobra.ExactArgs(0),
		RunE: func(c *cobra.Command, args []string) error {
......
			// 定义停止Channel信号
			stop := make(chan struct{})

			// 初始化服务
			discoveryServer, err := bootstrap.NewServer(serverArgs)
			if err != nil {
				return fmt.Errorf("failed to create discovery service: %v", err)
			}

			// 启动服务，带stop信号
			if err := discoveryServer.Start(stop); err != nil {
				return fmt.Errorf("failed to start discovery service: %v", err)
			}

            // 捕获中断信号
			cmd.WaitSignal(stop)
			// 执行清理动作
			discoveryServer.WaitUntilCompletion()
			return nil
		},
	}
......
```

1. 定义停止Channel信号，stop
2. 初始化服务，discoveryServer
3. 启动服务，带stop信号
4. 捕获中断信号，并更新stop信号
5. 执行清理动作

### 捕获中断信号

pkg/cmd/cmd.go:30

```go
func WaitSignal(stop chan struct{}) {
	sigs := make(chan os.Signal, 1)
	signal.Notify(sigs, syscall.SIGINT, syscall.SIGTERM)
	<-sigs
	close(stop)
	_ = log.Sync()
}
```

1. 捕获信号，syscall.SIGINT, syscall.SIGTERM
2. 更新信息，close(stop)

### 执行清理动作

pilot/pkg/bootstrap/server.go:388

```go
func (s *Server) WaitUntilCompletion() {
	s.requiredTerminations.Wait()
}
```

> `requiredTerminations sync.WaitGroup`，仅等待信号结束，结束动作由启动时注册好


注册带清理动作的任务：

pilot/pkg/bootstrap/server.go:689

```go
func (s *Server) addTerminatingStartFunc(fn startFunc) {
	s.addStartFunc(func(stop <-chan struct{}) error {
	    //+1
		s.requiredTerminations.Add(1)
		go func() {
		    //启动
			err := fn(stop)
			if err != nil {
				log.Errorf("failure in startup function: %v", err)
			}
			//-1
			s.requiredTerminations.Done()
		}()
		return nil
	})
}
```
