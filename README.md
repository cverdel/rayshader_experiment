# Rayshader experiment

![alt text][hermannsburg_plot]

[hermannsburg_plot]: https://github.com/cverdel/rayshader_experiment/blob/main/image3.png?raw=true

Trial script to make rayshader maps. This will produce a 3D render in an rgl window, as well as a 2D shaded relief map.

```
require(rayshader)
require(raster)
require(rgl)
require(rayrender)

#Load image
image_url<-"https://github.com/cverdel/rayshader_experiment/raw/main/Hermannsburg_map.tif"
temp<-tempfile()
download.file(image_url, temp, mode="wb")

rgb = raster::brick(temp)
raster::plotRGB(rgb, scale=255)
dim(rgb)

#Load elevation data
DEM_url<-"https://github.com/cverdel/rayshader_experiment/raw/main/Hermannsburg_DEM.tif"
temp2<-tempfile()
download.file(DEM_url, temp2, mode="wb")

elevation1 = raster::raster(temp2)
res(elevation1) #Resolution of a pixel
extent(elevation1) #Extent of raster
dim(elevation1) #Dimensions of raster

elevation<-aggregate(elevation1,fact=1)
res(elevation) #Resolution of a pixel
extent(elevation) #Extent of raster
dim(elevation) #Dimensions of raster

height_shade(raster_to_matrix(elevation)) %>%
  plot_map()

#Splits image up into rgb
names(rgb) = c("r","g","b", "a")
rgb_r = rayshader::raster_to_matrix(rgb$r)
rgb_g = rayshader::raster_to_matrix(rgb$g)
rgb_b = rayshader::raster_to_matrix(rgb$b)
rgb

#Check CRS
raster::crs(rgb)
raster::crs(elevation)

#Raster to matrix
el_matrix = rayshader::raster_to_matrix(elevation)

map_array = array(0,dim=c(nrow(rgb_r),ncol(rgb_r),3))

map_array[,,1] = rgb_r/255 #Red 
map_array[,,2] = rgb_g/255 #Blue 
map_array[,,3] = rgb_b/255 #Green 
map_array = aperm(map_array, c(2,1,3))

plot_map(map_array)

#Reduce the size of the elevation data, for speed
small_el_matrix = reduce_matrix_size(el_matrix, scale = 1) #Numbers less than 1 reduce the size of the elevation data

#Resize map to match elevation
resized_overlay_file = paste0(tempfile(),".png")
grDevices::png(filename = resized_overlay_file, width = dim(small_el_matrix)[1], height = dim(small_el_matrix)[2])
par(mar = c(0,0,0,0))
plot(as.raster(map_array))
dev.off()
overlay_img = png::readPNG(resized_overlay_file)

zscale=30 #Larger number makes less vertical exaggeration

#Render
ambient_layer = ambient_shade(small_el_matrix, zscale = zscale, multicore = TRUE, maxsearch = 200)
ray_layer = ray_shade(small_el_matrix, zscale = zscale, multicore = TRUE)

rgl::rgl.close() #Closes the rgl window

#Plot in 3D
(overlay_img) %>%
  add_shadow(ray_layer,0.3) %>%
  add_shadow(ambient_layer,0) %>%
  plot_3d(small_el_matrix,zscale=zscale)

#Plot in 2D
(map_array) %>%
  add_shadow(ray_layer,0.3) %>%
  add_shadow(ambient_layer,0.3) %>%
  plot_map()
```
