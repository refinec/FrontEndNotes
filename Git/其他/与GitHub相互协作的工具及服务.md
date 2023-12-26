# 与GitHub相互协作的工具及服务

* hub 命令**
  	封装了 git 命令的命令行工具，能够辅助用户使用 GitHub的工具
  	https://hub.github.com/
  	hub 命令将通常的 git 命令进行封装并增加几项功能，就可以调用 GitHub 的 API 发送命令
* **Travis CI**
  	http://travis-ci.org/
  	让 CI 软件监视仓库，可以在开发者发送提交后立刻执行自动测试或构建。通过持续执行这样一个操作，可以检测出开发者意外发送的提交或无意的逻辑偏差，让代码保持在一定质量以上
  	如果各位正在通过 GitHub 发布代码，建议使用 Travis CI。Travis CI 支持 Ruby、PHP、Perl、Python、Java、JavaScript 等
* **Coveralls**
  	代码覆盖率检测服务
  	https://coveralls.io/
  	Coveralls 可以为每一个 Pull Request 生成一份报告，我们建议各位使用这项服务，以时常提醒自己注意覆盖率问题。另外，由于用户可以通过详细报告了解哪些代码没有被测试，所以还有助用户改进自动测试的内容，提高测试效率。
* **Gemnasium**
  	https://gemnasium.com/
  	Gemnasium 服务可以查询 GitHub 仓库中软件正在使用的 RubyGems 或 npm（Node Package Manager，包管理器），让开发者了解自己是否正在使用最新版本进行开发
* **Code Climate**
  	代码分析报告服务 
  	这项服务可以分析 GitHub 仓库中的软件，查出软件中质量有问题的代码，同时给软件品质评级
  	https://codeclimate.com/
  	Code Climate 可以在我们的日常开发中分析代码，对容易出现 BUG 的复杂部分发出警告。如果不进行重构，与分析结果相伴的评级就会越来越低。这样一来可以督促我们在日常编写高品质代码，在评级下降时及时进行重构，让软件时常保持在一个高品质状态
* **Jenkins**
  	持续集成服务器
  	将把 GitHub 端仓库发来的 Pull Request 设置为触发器，让系统自动进行测试，并将测试结果发送至 GitHub。通过这种方法可以检验收到的 Pull Request 会不会破坏软件原有功能。另外，如果 Pull Request 会给某些功能带来 BUG 而无法通过测试，那么这个 Pull Request 将会像图 8.13 中那样显示在 GitHub 上，防止管理员误合并