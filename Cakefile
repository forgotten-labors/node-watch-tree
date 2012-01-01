fs   = require 'fs'
path = require 'path'
util = require 'util'
{exec} = require 'child_process'

option '-p', '--package [version]', 'Build provided version, according to npm\'s pack(1) command.'

task 'test', 'run tests', () ->
  async_testing = require 'async_testing'
  async_testing.run('lib/test/tests.js', [])


task 'build', 'src/ --> lib/', () ->
  # .coffee --> .js
  exec 'coffee -co lib src', (err, stdout, stderr) ->
    if err
      console.log stdout
      console.log stderr
      throw new Error "Error while compiling .coffee to .js"
      
task 'pack', 'Pack files into .tgz package', (opts) ->
  package = opts.package ? '.'
  console.time opts.arguments[0]
  console.log \
    if package is '.'
      "Packing files into .tgz package..."
    else
      "Building package from #{package}..."
  
  
  fs.mkdirSync('pkg', 0755) unless path.existsSync('pkg')
  files = fs.readdirSync(process.cwd()).filter (e, i, a) ->
    e if path.extname(e) is '.tgz'
  
  for file in files
    fs.renameSync(path.resolve(file), path.resolve(process.cwd(), "./pkg/#{file}"))
    
  watch = require('./lib/watch-tree').watchTree(process.cwd(), {'ignore': './pkg', 'match': /watch\-tree/})
  watch.on 'fileCreated', (p, s) ->  
    if path.extname(p) is '.tgz'
      filename = path.relative(process.cwd(), p)
      fs.renameSync(path.resolve(filename), path.resolve(process.cwd(), "pkg/#{filename}"))
      watch.removeAllListeners('fileCreated')
      console.timeEnd opts.arguments[0]
      process.exit(0)
    
  exec "npm pack #{package}", (e, out, err) ->
    if e
      console.log stdout
      console.log stderr
      throw new Error "Error while packing files to .tgz package"
    else
      