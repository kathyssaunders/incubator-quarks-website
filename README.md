<!--
{% comment %}
Licensed to the Apache Software Foundation (ASF) under one or more
contributor license agreements.  See the NOTICE file distributed with
this work for additional information regarding copyright ownership.
The ASF licenses this file to you under the Apache License, Version 2.0
(the "License"); you may not use this file except in compliance with
the License.  You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
{% endcomment %}
-->

# Quarks web site

This procedure was borrowed in part from the apex site. (https://git-wip-us.apache.org/repos/asf?p=incubator-apex-site.git) except we use jekyll.

  How it works
  ------------
 The master branch of this repo contains the source files that are used to generate the HTML that ultimately gets pushed to the incubator site.
The `asf-site` branch is where the actual generated files are stored. Note that this branch must contain exactly one folder called `content`,
 and so has been checked out as an orphan branch with its own commit history apart from the master branch. See the *Contributing* section below.
 
Through a [gitpubsub](http://www.apache.org/dev/gitpubsub.html) mechanism on the apache.org server,
files are taken from the `asf-site` branch and pushed to the live server.

Contributing
------------
If you would like to make a change to the site:
 
 1. Fork the [github mirror](https://github.com/apache/incubator-quarks-website)
 2. Create a new branch from `master`
 3. Add commit(s) to your branch
 4. Test your changes locally (see Developing)
 5. Open a pull request on the github mirror
 6. A committer will merge your changes if all is good 

If you are a committer, do the following:
  
 1. Update the master branch with your (or a Pull Request's) change.
 2. Push updated master to the asf remote master (https://git-wip-us.apache.org/repos/asf/incubator-quarks-site.git)
 3. Run `build.sh` from the master branch directory (requires jekyll). This will:
     - checks out and updates the `asf-site` branch with a new commit of the build from the current branch
 
 4. At this point, you should be on the `asf-site` branch. Simply push this branch to the asf remote and the site will automatically be updated within seconds.

Developing
-----------
 1. Make your changes under site
 2. cd site
 3. jekyll serve -d ../content_tmp
 4. point your browser to http://localhost:4000/


