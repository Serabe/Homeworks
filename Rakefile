require 'fileutils'
include FileUtils

ROOT_DIR = File.expand_path(File.dirname(__FILE__))

OUTPUT_DIR =File.join(ROOT_DIR, 'build')

mkdir_p OUTPUT_DIR unless File.exist?(OUTPUT_DIR)

SRC_DIR = File.join(ROOT_DIR, 'src')
SRC = Dir.glob(File.join(SRC_DIR,'**','*')).find_all{|x| File.file?(x)}

SRC_DOT = SRC.find_all{|x| File.extname(x) == '.dot'}
SRC_TEX = SRC.find_all{|x| File.extname(x) == '.tex'}

STYLE_DIR = File.join(ROOT_DIR, 'style')

TMP_DIR = File.join(ROOT_DIR, 'tmp')

def change_ext(file, new_ext)
  new_ext = '.'+new_ext unless new_ext[0,1]=='.'
  File.join(File.dirname(file),File.basename(file, File.extname(file)))+new_ext
end

mkdir_p TMP_DIR unless File.exist?(TMP_DIR)
[OUTPUT_DIR, TMP_DIR].each do |dir|
  def dir.for(file, ext=nil)
    ext = File.extname(file) unless ext
    File.join(self, File.basename(change_ext(file,ext)))
  end
end

class String
  def to_task()
    extmap = Hash.new {|hash, key| key}
    extmap.merge!({".tex" => ".pdf", ".dot" => ".png"})
    change_ext(self, extmap[File.extname(self)]).gsub(SRC_DIR+File::SEPARATOR,'').gsub('/',':')
  end

  def change_ns_to(new_ns)
    new_ns + self[(self.rindex(':'))..-1]
  end

  def ns
    self[0...(self.rindex(':'))]
  end
end

IMAGE_IN_TEX_REGEXP = /\\includegraphics(\[[^\]]*\])?\{([^\}]*\.(jpg|png|eps))\}/

SRC_TEX.each do |src|
  file src.to_task do
    cp src, TMP_DIR.for(src)
    sh "export TEXINPUTS=.:#{STYLE_DIR}:#{TMP_DIR}:#{OUTPUT_DIR}:$TEXINPUTS;pdflatex -output-directory #{OUTPUT_DIR} #{TMP_DIR.for(src)}"
  end
  
  File.read(src).scan(IMAGE_IN_TEX_REGEXP).each do |complete_match, file, ext, option|
    puts src.to_task + " => " + file.to_task.change_ns_to(src.to_task.ns)
    file src.to_task => file.to_task.change_ns_to(src.to_task.ns) if ext == 'png'
  end
end

SRC_DOT.each do |src|
  puts src.to_task
  file src.to_task do
    cp src, TMP_DIR.for(src)
    sh "dot -Tpng #{TMP_DIR.for(src)} -o #{TMP_DIR.for(src, '.png')}"
    cp TMP_DIR.for(src,'.png'), OUTPUT_DIR.for(src,'.png')
  end
end
