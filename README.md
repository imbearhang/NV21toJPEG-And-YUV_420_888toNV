# NV21toJPEG-And-YUV_420_888toNV
NV21toJPEG And YUV_420_888toNV

public static boolean saveImage(Image image, File file, boolean shouldCloseImage) {
        if (image == null)
            return false;
        byte[] nv21 = YUV_420_888toI420SemiPlanar(image.getPlanes()[0].getBuffer(), image.getPlanes()[1].getBuffer(), image.getPlanes()[2].getBuffer(), image.getWidth(), image.getHeight(), false);
        byte[] data = NV21toJPEG(nv21, image.getWidth(), image.getHeight(), 100);
        try {
            FileOutputStream output = new FileOutputStream(file);
            output.write(data);
            output.flush();
            output.close();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (shouldCloseImage) {
                image.close();
            }
        }
        return true;
    }


    public static byte[] NV21toJPEG(byte[] nv21, int width, int height, int quality) {
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        YuvImage yuv = new YuvImage(nv21, ImageFormat.NV21, width, height, null);
        yuv.compressToJpeg(new Rect(0, 0, width, height), quality, out);
        return out.toByteArray();
    }

    // nv12: true = NV12, false = NV21
    public static byte[] YUV_420_888toNV(ByteBuffer yBuffer, ByteBuffer uBuffer, ByteBuffer vBuffer, boolean nv12) {
        byte[] nv;

        int ySize = yBuffer.remaining();
        int uSize = uBuffer.remaining();
        int vSize = vBuffer.remaining();

        nv = new byte[ySize + uSize + vSize];

        yBuffer.get(nv, 0, ySize);
        if (nv12) {//U and V are swapped
            vBuffer.get(nv, ySize, vSize);
            uBuffer.get(nv, ySize + vSize, uSize);
        } else {
            uBuffer.get(nv, ySize, uSize);
            vBuffer.get(nv, ySize + uSize, vSize);
        }
        return nv;
    }

    public static byte[] YUV_420_888toI420SemiPlanar(ByteBuffer yBuffer, ByteBuffer uBuffer, ByteBuffer vBuffer,
                                                     int width, int height, boolean deInterleaveUV) {
        byte[] data = YUV_420_888toNV(yBuffer, uBuffer, vBuffer, deInterleaveUV);
        int size = width * height;
        if (deInterleaveUV) {
            byte[] buffer = new byte[3 * width * height / 2];

            // De-interleave U and V
            for (int i = 0; i < size / 4; i += 1) {
                buffer[i] = data[size + 2 * i + 1];
                buffer[size / 4 + i] = data[size + 2 * i];
            }
            System.arraycopy(buffer, 0, data, size, size / 2);
        } else {
            for (int i = size; i < data.length; i += 2) {
                byte b1 = data[i];
                data[i] = data[i + 1];
                data[i + 1] = b1;
            }
        }
        return data;
    }
