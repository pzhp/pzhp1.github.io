---
layout: post
title:  "TiDB SQL Execute Flow"
date:   2021-3-27 10:48:00
categories: TiDB,SQL,Design
---

# Some Points：

- 1）Session -> RecordSet -> Plan(LogicalPlan, PhysicalPlan) -> Executor
- 2）Pull Model： Volcano Model： Open/Next/Close
     Push Model： stream compute
- 3）About compute [decimal]:  
- 4）About Join:  HashJoin/Sort Merge Join/

```
session.Execute(ctx context.Context, sql string)  // (recordSets []sqlexec.RecordSet, err error)
	s.Parse(ctx, sql) 
	s.ExecuteStmt(ctx, stmtNodes[0])   // (sqlexec.RecordSet, error)
		compiler.Compile(ctx, stmtNode)  (*ExecStmt, error)
			plannercore.Preprocess(c.Ctx, stmtNode, infoSchema)
			planner.Optimize(ctx, c.Ctx, stmtNode, infoSchema)
				bestPlan, names, _, err := optimize(ctx, sctx, node, is)
					plannercore.NewPlanBuilder(sctx, is, hintProcessor)
					p, err := builder.Build(ctx, node)
					logic, isLogicalPlan := p.(plannercore.LogicalPlan)
					finalPlan, cost, err := plannercore.DoOptimize(ctx, sctx, builder.GetOptFlag(), logic)
						logic, err := logicalOptimize(ctx, flag, logic)
						physical, cost, err := physicalOptimize(logic, &planCounter)
						finalPlan := postOptimize(sctx, physical)
		recordSet, err := runStmt(ctx, s, stmt) 
			rs, err = s.Exec(ctx)  // (rs sqlexec.RecordSet, err error) 
				e, err := a.buildExecutor()  // a is ExecStmt
					b := newExecutorBuilder(ctx, a.InfoSchema)
					e := b.build(a.Plan)
				err = e.Open(ctx)
![image](https://user-images.githubusercontent.com/5364483/112707989-335a9d00-8eea-11eb-9c33-b9e92ee8eed3.png)
```


[decimal]:https://www.imhanjm.com/2017/08/27/go%E5%A6%82%E4%BD%95%E7%B2%BE%E7%A1%AE%E8%AE%A1%E7%AE%97%E5%B0%8F%E6%95%B0-decimal%E7%A0%94%E7%A9%B6/


