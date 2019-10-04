---


---

<h1 id="hello-docker">Hello Docker!!</h1>
<p>Hands on exploration of docker capabilities:</p>
<h3 id="running-the-first-docker-container">Running the first docker container</h3>
<pre><code>docker run hello-world
</code></pre>
<h3 id="fetching-the-docker-image">Fetching the docker image</h3>
<p>Listing the current images on the local vm</p>
<pre><code>docker images 
</code></pre>
<p>Pulling the busybox image</p>
<pre><code>docker pull busybox
</code></pre>
<p>Listing the images on the local vm again, now we should find the busybox image</p>
<pre><code>docker images 
</code></pre>
<p>So how is it different from virtual machines</p>
<p><img src="https://docs.docker.com/images/VM@2x.png" alt="Virtual Machine" width="450"></p>
<p><img src="https://docs.docker.com/images/Container@2x.png" alt="Docker Instance" width="450"></p>
<h3 id="running-the-busybox-container">Running the busybox container</h3>
<pre><code>docker run -it busybox sh
</code></pre>
<blockquote>
<p><strong>Note:</strong> The flags represented are<br>
-i -&gt; interactive shell<br>
-t -&gt; Allocate a pseudo-TTY<br>
More run options can be found <a href="https://docs.docker.com/engine/reference/commandline/run/">here</a></p>
</blockquote>
<p>Find the container IP address</p>
<pre><code>ip address
</code></pre>
<p><img src="https://docs.docker.com/engine/tutorials/bridge1.png" alt="Docker networking" width="350"></p>
<p>Run the echo command passed to docker run</p>
<pre><code>docker run -it busybox echo  "hello from busybox"
</code></pre>
<h3 id="loading-a-set-of-environment-variables">Loading a set of environment variables</h3>
<pre><code>docker run --env VAR1=value1 --env VAR2=value2 busybox env | grep VAR
</code></pre>
<h3 id="mounting-a-device-to-the-container">Mounting a device to the container</h3>
<pre><code>$ docker run --device=/dev/sda:/dev/xvdc --rm -it busybox fdisk  /dev/xvdc

Command (m for help): q
</code></pre>
<h3 id="difference-between-privileged-mode-and-un-privileged-mode">Difference between privileged mode and un-privileged mode</h3>
<pre><code>$ docker run -t -i --rm ubuntu bash
root@0ad207e60437:/# mount -t tmpfs none /mnt
mount: /mnt: permission denied.
</code></pre>
<pre><code>$ docker run -t -i --privileged ubuntu bash
root@81e58131300a:/# mount -t tmpfs none /mnt
root@81e58131300a:/# df -h
Filesystem      Size  Used Avail Use% Mounted on
none            1.9G     0  1.9G   0% /mnt
</code></pre>
<h3 id="binding-a-port-address">Binding a port address</h3>
<pre><code>docker run -d -p 8080:80 --name static-site nginx
</code></pre>
<pre><code>$ docker run -d -P --name static-site2 nginx

$ docker ps
</code></pre>
<h4 id="listing-all-containers-currently-running">Listing all containers currently running</h4>
<pre><code>docker ps
</code></pre>
<p>Listing all containers</p>
<pre><code>docker ps -a
</code></pre>
<h4 id="to-stop-all-the-containers">To stop all the containers</h4>
<pre><code>docker stop $(docker ps -q)
</code></pre>
<h4 id="to-delete-the-containers">To delete the containers</h4>
<pre><code>docker rm $(docker ps -aq -f status=exited)
</code></pre>
<h2 id="building-a-docker-image">Building a docker image</h2>
<p>Create a <strong>Dockerfile</strong> in a new folder</p>
<pre class=" language-dockerfile"><code class="prism  language-dockerfile"><span class="token comment"># Use an official Python runtime as a parent image</span>
<span class="token keyword">FROM</span> python<span class="token punctuation">:</span>2.7<span class="token punctuation">-</span>slim

<span class="token comment"># Set the working directory to /app</span>
<span class="token keyword">WORKDIR</span> /app

<span class="token comment"># Copy the current directory contents into the container at /app</span>
<span class="token keyword">COPY</span> . /app

<span class="token comment"># Install any needed packages specified in requirements.txt</span>
<span class="token keyword">RUN</span> pip install <span class="token punctuation">-</span><span class="token punctuation">-</span>trusted<span class="token punctuation">-</span>host pypi.python.org <span class="token punctuation">-</span>r requirements.txt

<span class="token comment"># Make port 80 available to the world outside this container</span>
<span class="token keyword">EXPOSE</span> 80

<span class="token comment"># Define environment variable</span>
<span class="token keyword">ENV</span> NAME World

<span class="token comment"># Run app.py when the container launches</span>
<span class="token keyword">CMD</span> <span class="token punctuation">[</span><span class="token string">"python"</span><span class="token punctuation">,</span> <span class="token string">"app.py"</span><span class="token punctuation">]</span>
</code></pre>
<p>Create a <code>requirements.txt</code> file with this content:</p>
<pre><code>Flask
Redis
</code></pre>
<p>create another file  <code>app.py</code> with this content:</p>
<pre class=" language-python"><code class="prism  language-python"><span class="token keyword">from</span> flask <span class="token keyword">import</span> Flask
<span class="token keyword">from</span> redis <span class="token keyword">import</span> Redis<span class="token punctuation">,</span> RedisError
<span class="token keyword">import</span> os
<span class="token keyword">import</span> socket

redis <span class="token operator">=</span> Redis<span class="token punctuation">(</span>host<span class="token operator">=</span><span class="token string">"redis"</span><span class="token punctuation">,</span> db<span class="token operator">=</span><span class="token number">0</span><span class="token punctuation">,</span> socket_connect_timeout<span class="token operator">=</span><span class="token number">2</span><span class="token punctuation">,</span> socket_timeout<span class="token operator">=</span><span class="token number">2</span><span class="token punctuation">)</span>

app <span class="token operator">=</span> Flask<span class="token punctuation">(</span>__name__<span class="token punctuation">)</span>

@app<span class="token punctuation">.</span>route<span class="token punctuation">(</span><span class="token string">"/"</span><span class="token punctuation">)</span>
<span class="token keyword">def</span> <span class="token function">hello</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">:</span>
	<span class="token keyword">try</span><span class="token punctuation">:</span>
		visits <span class="token operator">=</span> redis<span class="token punctuation">.</span>incr<span class="token punctuation">(</span><span class="token string">"counter"</span><span class="token punctuation">)</span>
	<span class="token keyword">except</span> RedisError<span class="token punctuation">:</span>
		visits <span class="token operator">=</span> <span class="token string">"&lt;i&gt;cannot connect to Redis, counter disabled&lt;/i&gt;"</span>

	html <span class="token operator">=</span> <span class="token string">"&lt;h3&gt;Hello {name}!&lt;/h3&gt;&lt;b&gt;Hostname:&lt;/b&gt; {hostname}&lt;br/&gt;&lt;b&gt;Visits:&lt;/b&gt; {visits}"</span>
	<span class="token keyword">return</span> html<span class="token punctuation">.</span><span class="token builtin">format</span><span class="token punctuation">(</span>name<span class="token operator">=</span>os<span class="token punctuation">.</span>getenv<span class="token punctuation">(</span><span class="token string">"NAME"</span><span class="token punctuation">,</span> <span class="token string">"world"</span><span class="token punctuation">)</span><span class="token punctuation">,</span> hostname<span class="token operator">=</span>socket<span class="token punctuation">.</span>gethostname<span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">,</span> visits<span class="token operator">=</span>visits<span class="token punctuation">)</span>

<span class="token keyword">if</span> __name__ <span class="token operator">==</span> <span class="token string">"__main__"</span><span class="token punctuation">:</span>
	app<span class="token punctuation">.</span>run<span class="token punctuation">(</span>host<span class="token operator">=</span><span class="token string">'0.0.0.0'</span><span class="token punctuation">,</span> port<span class="token operator">=</span><span class="token number">80</span><span class="token punctuation">)</span>
</code></pre>
<p>Build the docker image</p>
<pre><code>$ docker build --tag=friendlyhello .

$ docker image ls

$ docker run -d -p 4000:80 friendlyhello
</code></pre>
<p><img src="https://miro.medium.com/max/450/1*kTDjPNUqGX8ZdLidbukheA.png" alt="Docker Image Layers"></p>
<p>Finding the different layers in the created image</p>
<pre><code>docker history friendlyhello
</code></pre>
<h3 id="user-defined--bridge-networks">User defined  bridge networks</h3>
<pre><code>$ docker network create -d bridge --subnet 10.0.0.0/24 my_bridge

$ docker run --rm -itd --name c2 --net my_bridge busybox sh
</code></pre>
<p>Using a specific IP from the created network</p>
<pre><code>$ docker run --rm -it --name c3 --net my_bridge --ip 10.0.0.254 busybox sh
# ip a
# ping c2
# exit
</code></pre>
<p>Checking the c2 IP address</p>
<pre><code>$ docker exec -ti c2 sh
# ip a
</code></pre>

