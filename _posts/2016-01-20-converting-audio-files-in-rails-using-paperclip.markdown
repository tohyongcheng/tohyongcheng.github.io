---
layout: post
comments: true
title:  "Converting audio files in Rails using Paperclip"
date:   2016-01-20 11:00:00
categories: "ruby on rails"
---

You can use Paperclip Processors to convert audio files in Rails. I was using Rails to receive .mp3 files and converting them to .m4a and .amr so that iOS devices and Android devices can play the respective audio files. You have to first install ```ffmpeg``` in the system that you have, and oh dear... It can really be quite a pain in installing ffmpeg. I had a hard time installing on my Ubuntu machine because of all the dependencies which I needed. I forgot how I did it, but more information on how to install it can be found at https://trac.ffmpeg.org/wiki/CompilationGuide/MacOSX

Luckily for Mac users with homebrew, you can just install by using the following command:

```
brew install ffmpeg --with-fdk-aac --with-ffplay --with-freetype --with-libass --with-libquvi --with-libvorbis --with-libvpx --with-opus --with-x265
```

We will be using ffmpeg mainly to convert our audio files. To convert .mp3 file to .amr, you can use do:
```
$ ffmpeg -i audio1.mp3 -ar 22050 audio1.amr
```

This converts an input named ```audio1.mp3``` to ```audio1.amr``` with an audio rate of 22.05Khz.

Now, it's time to use Rails to make full use of ```ffmpeg```. First, setup a model that contains an audio file with paperclip:

{% highlight ruby %}
class AudioMessage < ActiveRecord::Base

  has_attached_file :audio,
                    :styles => lambda { |attachment|
                      {
                        m4a: { :processors => [:m4a_processor] },
                        amr: { :processors => [:amr_processor] },
                      }
                    }
end
{% endhighlight %}

In this model, we have an AudioMessage ActiveRecord model. We are using Paperclip to store the file ```audio```. We have set the .mp3 files to be converted to .m4a format via the m4a\_processor, and the .amr format via the .amr\_processor. Now it's time to setup the processors. We will need to create a file for each processor in ```/lib/paperclip_processors```. Paperclip automatically uses this directory to look for its processor files. Let's start by creating our ```amr_processor```.

{% highlight ruby %}
# lib/paperclip_processors/amr_processor.rb
module Paperclip
  class AmrProcessor < Processor
    def initialize(file, options = {}, attachment = nil)
      super
    end
  end
end
{% endhighlight %}


The AmrProcessor class we use here subclasses from the Processor class provided by Paperclip. These Processor class allows you to modify the files that you attach to the model, and does not only apply to audio. You can use it for image processing such as cropping or inserting watermarks. For the initialization, we use ```super``` to inherit the default behaviour from ```Paperclip::Processor```.

Next we need to make some attributes accessible for the class.

{% highlight ruby %}
def initialize(file, options = {}, attachment = nil)
  super

  # make the file and attachment accessible to the module and class
  @file = file
  @attachment = attachment

  # set some default parameters for basename, eventual file format
  @basename = File.basename(@file.path)
  @format = options[:format] || 'amr'
  
  # -y is to allow ffmpeg to overwrite output without asking, and -i is to denote the input filename for the next argument
  @params = options[:params] || '-y -i' 

  # set some default sample_rate and bit_rate which we will use for the conversion
  @sample_rate = options[:sample_rate] || "8000"
  @bit_rate = options[:bit_rate] || "12.2k"
end
{% endhighlight %}

All these will be used to determine the command we will use to run our ffmpeg command. It is now time to fill in the ```make``` method which is provided by Paperclip::Processor but will need to be modified to give the behaviour of converting our audio file.

{% highlight ruby %}
module Paperclip
  class AmrProcessor < Processor
    [...]

    def make
      source = @file
      output = Tempfile.new([@basename, ".#{@format}"]) 
      begin
        parameters = [@params, ':source', '-ar' ,':sample_rate', '-ab', ':bit_rate', ':dest'].join(' ')
        Paperclip.run('ffmpeg', parameters, :source => File.expand_path(source.path), :dest => File.expand_path(output.path), :sample_rate => @sample_rate, :bit_rate => @bit_rate)
      rescue PaperclipCommandLineError
        raise PaperclipError, "There was an error converting #{@basename} to .amr"
      end
      output
    end
  end
end
{% endhighlight %}

First, we declare our source file, and the output that we want. Note that ```@format``` here is ```amr```, so that the eventual output will be in .amr format. The ```parameters``` term is used to run alongside with ffmpeg through ```Paperclip.run()``` method where you can fill in interpolation values in the string that we have in ```parameters```:

```parameters = "-y -i :source -ar :sample_rate -ab :bit_rate :dest"```

```Paperclip.run()``` eventually will run the ffmpeg command stated near the beginning of this post, with all the sample rate and bitrates determined by our parameters in the ```initialize``` method. The named arguments we have in Paperclip.run will substitute the symbol strings we have in ```parameters``` to run the command such as:

```
ffmpeg -y -i source.mp3 -ar 8000 -ab 12.2k destination.amr
```

With this, your processor is set! The audio conversion should happen immediately after you upload an audio file tagged to the AudioMessage object. Do the same for the M4aProcessor, and you will be able to convert the uploaded file into both .amr and .m4a formats automatically. 

M4aProcessor:
{% highlight ruby %}
# lib/paperclip_processors/m4a_processor.rb
module Paperclip
  class M4aProcessor < Processor
    def initialize(file, options = {}, attachment = nil)
      super
      @file = file
      @attachment = attachment
      @params = options[:params] || '-y -i'
      @basename = File.basename(@file.path)
      @format = options[:format] || 'm4a'
      @sample_rate = options[:sample_rate] || "8000"
      @bit_rate = options[:bit_rate] || "12.2k"
    end

    def make
      source = @file
      output = Tempfile.new([@basename, ".#{@format}"])
      begin
        parameters = [@params, ':source', ':dest'].flatten.compact.join(' ').strip.squeeze(' ')
        Paperclip.run('ffmpeg', parameters, :source => File.expand_path(source.path), :dest => File.expand_path(output.path), :sample_rate => @sample_rate, :bit_rate => @bit_rate)
      rescue PaperclipCommandLineError
        raise PaperclipError, "There was an error converting #{@basename} to m4a"
      end
      output
    end
  end
end
{% endhighlight %}


