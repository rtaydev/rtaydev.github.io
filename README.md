# Neurodivergent Developer

## Mission Statement

Our mission is to raise awareness, provide resources, and offer support for neurodivergent individuals, especially developers, to help them access assessments, community, and professional support.

## About the Author

Principal Engineer | React Native, Swift, Kotlin | 20+ years in mobile app development, leading major projects. Open to new opportunities and collaborations. Recently diagnosed with AuADHD, neurodivergent.

## How to Setup, Run, and Deploy a Jekyll Blog for GitHub Pages

### Setup

1. **Install Ruby**: Ensure you have Ruby installed on your system. You can download it from [ruby-lang.org](https://www.ruby-lang.org/en/downloads/).
2. **Install Jekyll and Bundler**: Open your terminal and run:

   ```sh
   gem install jekyll bundler
   ```

3. **Create a new Jekyll site**: Navigate to the directory where you want to create your site and run:

   ```sh
   jekyll new myblog
   ```

4. **Navigate to your new site directory**:

   ```sh
   cd myblog
   ```

### Run

1. **Install dependencies**: Run the following command to install the dependencies:

   ```sh
   bundle install
   ```

2. **Serve the site**: Run the following command to build and serve the site locally:

   ```sh
   bundle exec jekyll serve
   ```

3. **View your site**: Open your browser and go to `http://localhost:4000` to see your site.

### Deploy

1. **Create a GitHub repository**: Go to GitHub and create a new repository named `myblog`.
2. **Push your site to GitHub**:

   ```sh
   git init
   git add .
   git commit -m "Initial commit"
   git remote add origin https://github.com/yourusername/myblog.git
   git push -u origin master
   ```

3. **Configure GitHub Pages**:
   - Go to your repository on GitHub.
   - Click on `Settings`.
   - Scroll down to the `GitHub Pages` section.
   - Under `Source`, select `master branch` or `main branch`.
   - Click `Save`.

Your Jekyll blog should now be live on GitHub Pages at `https://yourusername.github.io/myblog`.
