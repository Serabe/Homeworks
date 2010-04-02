require 'FileUtils'
include FileUtils

ROOT_DIR = File.expand_path(__FILE__)

OUTPUT_DIR =File.join(ROOT_DIR, 'build')

mkdir_p OUTPUT_DIR

SRC_DIR = File.join(ROOT_DIR, 'src')
SRC = Dir.new(SRC_DIR).find_all{|x| File.file?(x)}
SRC_TEX = SRC.find_all{|x| File.extname(x) == '.tex'}

STYLE_DIR = File.join(ROOT_DIR, 'style')

TMP_DIR = File.join(ROOT_DIR, 'tmp')

mkdir_p TMP_DIR

def TMP_DIR.for(file)
  File.join(TMP_DIR, File.basename(file))
end

IMAGE_IN_TEX_REGEXP = /\\includegraphics(\[h\])?\{([^\}]*\.(jpg|png|eps))\}/

SRC_TEX.each do |src|
  file src do
    cp src, TMP_DIR.for(src)
    sh "export TEXINPUTS=.:#{STYLE_DIR}:#{TMP_DIR};pdflatex #{TMP_DIR.for(src)} -output-directory #{OUTPUT_DIR}"
  end
  
  File.read(src).scan(IMAGE_IN_TEX_REGEXP).each do |complete_match, option, file, ext|
    file src => file
    file file do
      cp file, TMP_DIR.for(file)
      sh "dot -T#{ext} #{TMP_DIR.for(file)}"
    end
  end
end