# Docker and Cloud Foundry

> 这篇文章写于April 8, 2014. 作者是Phil Whelan(菲尔·惠兰), 任职ActiveState公司软件设计师. 从事私有云Stackato的研发.

In this post I will give an overview of where you might find Docker integration within the Cloud Foundry ecosystem. We will look at Decker, Stackato and Diego.

> 开篇介绍在CF生态系统中集成Docker的3个成功案例。当前，CF V2并没有使用Docker作为App的运行容器，而是使用的自行研发的Warden容器。CF团队已经计划使用Docker容器，目前正处于孵化期阶段，就是后面要介绍的Diego。你可以在[这里](https://github.com/cloudfoundry-incubator/)找到CF处于孵化阶段的组件。

## Decker

Last week I was lucky to make it to the [London PaaS User Group meetup](http://www.meetup.com/London-PaaS-User-Group-LOPUG/). There I met Colin Humphreys of CloudCredo and saw him give a demo of [Decker](http://www.cloudcredo.com/decker-docker-cloud-foundry/).

> 科林·汉弗莱斯, CloudCredo公司软件设计师。CloudCredo公司是一家针对CF平台提供安装、咨询等服务的小型公司。

Decker is a prototype that Colin has built to use Docker as a backend to Cloud Foundry. Decker implements the DEA API, so it is a drop-in component to Cloud Foundry. This is similar to how other 3rd party DEAs, such as [Iron Foundry](http://www.ironfoundry.org/), have been implemented.

> CF原生DEA使用的是Warden容器,而Decker是使用Docker容器的DEA, 它实现了DEA所有的API.

> Iron Foundry，另一个DEA实现，用以支持.NET应用。这个项目也是开源的，维护在[GitHub](http://www.github.com/ironfoundry)上。该项目是由一家小型云厂商Tier 3发起的，该厂商于2013年11月被CenturyLink收购。

Cloud Foundry's HTTP API plus NATS message bus protocol enables anyone to write a plug-in component to stage and run application instances. This also means you can run applications on top of non-Linux platforms, such as Windows, while still running Cloud Foundry on Linux - just a long as the 3rd party DEA talks to the rest of the system using the Cloud Foundry DEA protocol. This is what Colin has done with Decker.

> 这段话意思就是说，有的组件（程序）是可以运行在其他操作系统上的，并不一定都是运行在Linux系统上。这也是开发Decker的一个目标。为什么这么说呢？目前大部分App容器都是基于LXC来开发的，这些特性是只有某些特定版本的Linux才原生支持的。

When you wish to deploy an application to Docker, you specify ``--stack decker`` with your ``cf push`` command. The Decker DEA advertises that it supports this stack and so the Cloud Controller will direct the application instances there.

> 如何将App部署到Docker中. 因Decker组件在初始运行时已经声明，所以Cloud Controller能够直接将App部署进去。

> 我特意咨询了下本文的作者, 确认Deckor是另一个DEA实现模块, 在CF中可以与原生DEA共存, 使用``cf push --stack deckor``可以将App部署进Deckor中. 

Decker currently works with [Dockerfiles](http://docs.docker.io/en/latest/reference/builder/), rather than with built Docker images. A Dockerfile is a single file that lists all the commands that are run to build a Docker image.

> Decker使用Dockerfiles，而不是使用已构建好的Docker镜像。Dockerfile就是用来创建Docker镜像的。

When ``cf push`` is run, the Dockerfile is uploaded to the Cloud Controller. The Cloud Controller selects Colin's Decker DEA for deployment, due to the ``--stack decker`` being specified.

As Colin admits, the staging process of the Decker prototype is a bit of cheat right now and does not not actually do much in the way of staging. Currently, Decker will simply persist the Dockerfile in the droplet and leave the building of the Docker image until runtime. With each instance of the droplet that is deployed, Decker will extract the Dockerfile from the droplet, build the Docker image and then run the built Docker image as a new container.

> 目前为止，Decker并没有将打包（Stage）过程中将Dockerfile转换成Docker镜像，而仅仅是将Dockerfile置入Droplet，而是等到运行期（Droplet进入App容器后解压运行）才进行生成Docker镜像动作。而且，每个Droplet实例进行部署，都会进行Docker镜像生成动作。

Obviously, building new Docker images at runtime is inefficient and may lead to "[snowflake](http://martinfowler.com/bliki/SnowflakeServer.html)" instances if any of the dependencies change or anything unusual happens at Docker image build time. As Colin mentioned, this is an MVP (minimum viable product) and this will be addressed in future iterations.

> 很明显这样做有性能问题。不过文中也指出这仅是一个MVP（最简可行产品, 类似原型开发）, 后期迭代中将得到解决。

Security is currently a concern with allowing Cloud Foundry users to deploy any Docker image of their choosing. When you are allowed to build your own images it is quite easy to allow the user of that container to run as root. cgroups' user-namespacing is based on the user id of the host system. Users inside the container will be given a unique id that does not exist outside the container. Unfortunately, the ``root`` user has id of ``0`` both inside and outside of the container, so it is the same user both inside and outside of the container. If a user becomes ``root`` inside the container, then there is potential for them to break out of the container and gain access to the host system.

> 允许用户部署他们选择的Docker镜像，这是一个值得关注的安全问题。因``root``用户在容器内外都具有相同的id(``0``)。所以，一旦在容器内用户变成``root``，那么就有可能打破容器直接访问宿主系统。

## Stackato

Last December, ActiveState released [Stackato 3.0](http://www.activestate.com/blog/2013/11/technical-look-stackato-v30-beta) in which we replaced our own LXC implementation with Docker. We had several years experience of working with LXC and cgroups and analyzed dotCloud's open-sourced implementation, called Docker. We liked the way they had implemented it and saw Docker's obvious potential in the future of PaaS, but also knew the line we needed to draw around our integration. Docker was young, not recommended for production, but the basics of provisioning LXC containers was solid.

> ActiveState公司的Stackato是在CF基础上开发的私有云产品。在3.0版本中使用了Docker容器。因为CF v2使用的Warden容器是利用cgroup机制的，并没有使用LXC，所以可以看出Stackato早期的版本也没有使用Warden容器。看来Warden技术很没前途，前景堪忧啊！文中提到Docker技术尚显不足，不建议用于商业用途，但提供LXC容器这基本功能还很坚实，所以在PaaS上还是前途无量的！

Therefore, in Stackato 3.0 the integration with Docker was minimal. We replaced Stackato's singular LXC template with a single Docker base image. The provisioned containers did not have sudo access (by default) and there was no way for the application developer to specify how the Docker image was built in terms of Docker functionality. As far as Stackato was concerned, fence (Stackato's LXC container manager) was still just provisioning basic LXC containers. The difference was that the LXC provisioning now had [ten thousand eyeballs](https://github.com/dotcloud/docker/stargazers) on it and ActiveState had laid the groundwork for Stackato to be the enterprise-grade Docker PaaS when Docker reaches maturity.

> Stackato 3.0仅集成了Docker最小的基本功能，并且限制了一些功能。

Application staging in Stackato 3.0, similar to Cloud Foundry v2, became buildpack centric. To the end-user this may not have been entirely obviously since we now have built-in "legacy buildpacks" which replicate the behavior of Stackato 2.10's (and prior) resident support for runtimes and frameworks.

When Stackato's container management daemon, fence, provisions the LXC container via the base Docker image, the LXC container is used to build up the stack using buildpacks and [staging hooks](http://docs.stackato.com/user/deploy/stackatoyml.html#hooks). Droplets are then extracted from the LXC container in the same way as is was in pre-3.0 Stackato.

It is possible for a Stackato administrator to [change the base Docker image](http://docs.stackato.com/3.2/admin/server/docker.html) that fence uses with a few kato commands.

```
$ kato config get fence docker/image
stackato/stack/alsek
$ kato config set fence docker/image exampleco/newimg
exampleco/newimg
```

We still have not gone "fully Docker" for good reasons. It still is not safe for us to let users deploy Docker images where they may be able to gain root access to the container and subsequently to the host system. This has been solved at the LXC/cgroups level and as we speak I am sure somebody is working on the Docker implementation, but it is not available yet. We are also maintaining Ubuntu LTS and supported kernels, so we are waiting for the all the stars to align.

> 之所以没使用全部功能的Docker，还是因为安全原因。在LXC/cgroup级别是可以解决问题的，有人正在做这件事，尚未公布，观望中...

## Diego

Diego is a new component of Cloud Foundry which aims to re-architect the way that staging and deployment is managed. This essentially replaces the DEA with something that should be more extensible across a variety of runtime environments.

> Diego是CF新组件，目前正处于孵化阶段。提供了一种新的打包和部署方式，同时也可以替代DEA的运行环境功能。

For a long time Cloud Foundry has used Warden for its container management. Similar to Docker, or Stackato's original LXC implementation, Warden is based on LXC and cgroups. With Diego, Warden becomes Garden.

So where does Docker fit into Diego?

The integration is minimal and you will not find Docker in a deployed Cloud Foundry cluster. Docker is currently only used to generate the LXC template that Garden then uses. Docker is simply a build-time tool for the Cloud Foundry release.

> 目前Diego只使用了Docker很少一部分, 仅用来生成LXC模板(模板是给Garden用的), 现在Docker仅是个build-time工具.

Although, it is possible that Diego could open up the way to provide an optional Docker backend.

Diego has 3 components - the Stager, the Smelter and the Executor. First the Stager, which runs on Linux, sets up the job for smelting. The Smelter, running on the target platform, creates the application droplet. It is then the job of the Executor, which also runs on the target platform, to run the droplet as a running application.

> Diego有3个组件,各司其职. 分别用于创建任务, 创建Droplet, 运行Droplet.

The Smelter and Executor, which run on the target platform, provide a way to support any backend, whether it be Linux, Windows or other. The Linux backend is provided by Garden (formerly Warden). It should be possible to have another, albeit redundant, Linux backend, such as Docker.

> Smelter和Executor组件进行更高一层的抽象化,设计为可运行在任何平台上:linux,windows或其它. 目前的支持linux后台的实现是通过Garden(原先为Warden).当然也有可能支持另一个Linux实现:Docker.

##Conclusion

There is great potential for Docker and Cloud Foundry collaboration. Integration has been proved with 3 different projects. Decker, Stackato and Diego each taking a different approach to the integration points. Obviously Diego's integration currently lives outside of a built Cloud Foundry release, but there is further potential for providing Docker integration via the Smelter and Executor.

> 目前Diego还不在CF发行版中,但是将来有可能通过Smelter和Executor来整合Docker.

Colin Humphreys made an interested point at the London PaaS User Group last week. He said that he thinks that the current model of PaaS should be split so that we have another layer. This layer he calls CaaS, or Containers-as-a-Service. This idea is the motivation behind Decker.

> 新观点: 从PaaS层再分离出一个层级:CaaS(Containers-as-a-Service)

How closely integrated should a PaaS be with its containerization implementation? Should Cloud Foundry users be able to easily plug in and out different container managers, such as switching out Warden for Docker? What do we lose from the overhead of decoupling these? What do we gain?