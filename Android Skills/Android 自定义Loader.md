---
title: Android 自定义Loader
date: 2014-05-17 22:01:14
tags: 自定义Loader
categories: 技术
---

### 自定义Loader获取本机照片

####  图片实体类（ImageBean）

<!--more-->
    public class ImageBean implements Parcelable {

        private String path = null;
        private boolean isSeleted = false;

        public ImageBean(String path, boolean selected) {
            this.path = path;
            this.isSeleted = selected;
        }

        public ImageBean() {
        }

        @Override
        public int describeContents() {
            return 0;
        }

        @Override
        public void writeToParcel(Parcel dest, int flags) {
            dest.writeString(path);
            dest.writeInt(isSeleted ? 1 : 0);
        }

        public static final Creator<ImageBean> CREATOR = new Creator<ImageBean>() {

            @Override
            public ImageBean createFromParcel(Parcel source) {
                return new ImageBean(source.readString(), source.readInt() == 1 ? true : false);
            }

            @Override
            public ImageBean[] newArray(int size) {
                return new ImageBean[size];
            }
        };

        /**
         * @return the path
         */
        public String getPath() {
            return path;
        }

        /**
         * @param path the path to set
         */
        public void setPath(String path) {
            this.path = path;
        }

        /**
         * @return the isSeleted
         */
        public boolean isSeleted() {
            return isSeleted;
        }

        /**
         * @param isSeleted the isSeleted to set
         */
        public void setSeleted(boolean isSeleted) {
            this.isSeleted = isSeleted;
        }
    }


#### 自定义读取图片的Loader（ImagesLoader）
-
    public class ImagesLoader extends AsyncTaskLoader<ArrayList<ImageBean>> {

        private ArrayList<ImageBean> mImages = null;

        /**
         * @param context
         */
        public ImagesLoader(Context context) {
            super(context);
        }

        @Override
        public ArrayList<ImageBean> loadInBackground() {
            ArrayList<ImageBean> imageList = new ArrayList<ImageBean>();
            Cursor imageCursor = getContext().getContentResolver().query(MediaStore.Images.Media.EXTERNAL_CONTENT_URI, new String[]{MediaStore.Images.Media.DATA, MediaStore.Images.Media._ID}, null, null, MediaStore.Images.Media._ID);
            if (imageCursor != null && imageCursor.getCount() > 0) {
                while (imageCursor.moveToNext()) {
                    ImageBean item = new ImageBean(imageCursor.getString(imageCursor.getColumnIndex(MediaStore.Images.Media.DATA)), false);
                    imageList.add(item);
                }
            }
            if (imageCursor != null) {
                imageCursor.close();
            }
            // show newest photo at beginning of the list
            Collections.reverse(imageList);
            return imageList;
        }

        /* Runs on the UI thread */
        @Override
        public void deliverResult(ArrayList<ImageBean> images) {
            if (isReset()) {
                // An async query came in while the loader is stopped
                if (images != null) {
                    images.clear();
                    images = null;
                }
                return;
            }
            ArrayList<ImageBean> oldImages = mImages;
            mImages = images;

            if (isStarted()) {
                super.deliverResult(images);
            }

            if (oldImages != null && oldImages != mImages) {
                oldImages.clear();
                oldImages = null;
            }
        }

        @Override
        protected void onStartLoading() {
            if (mImages != null && mImages.size() > 0) {
                deliverResult(mImages);
            }
            if (takeContentChanged() || mImages == null) {
                forceLoad();
            }
        }

        /**
         * Must be called from the UI thread
         */
        @Override
        protected void onStopLoading() {
            // Attempt to cancel the current load task if possible.
            cancelLoad();
        }

        @Override
        public void onCanceled(ArrayList<ImageBean> images) {
            if (images != null) {
                images.clear();
                images = null;
            }
        }

        @Override
        protected void onReset() {
            super.onReset();
            // Ensure the loader is stopped
            onStopLoading();
            if (mImages != null) {
                mImages.clear();
                mImages = null;
            }
        }

    }

    
#### Activity

    public class GalleryActivity extends AppCompatActivity implements LoaderManager.LoaderCallbacks<ArrayList<ImageBean>>  {
        ......
        ......
        ......
        @Override
        public Loader<ArrayList<ImageBean>> onCreateLoader(int i, Bundle bundle) {
            return new ImagesLoader(this);
        }

        @Override
        public void onLoadFinished(Loader<ArrayList<ImageBean>> loader, ArrayList<ImageBean> images) {
            this.mImages = images;   
        }

        @Override
        public void onLoaderReset(Loader<ArrayList<ImageBean>> images) {

        }
    }

### 自定义Loader获取联系人

#### 联系人实体（ContactInfo）

    public class ContactInfo implements Serializable {
        private String name;
        private String phoneNumber;
        private boolean selected;
        private String pinyinTag;
        private String pinyin;

        public ContactInfo() {
        }

        public ContactInfo(String name, String phoneNumber, String pinyinTag, String pinyin) {
            this.setName(name);
            this.setPhoneNumber(phoneNumber);
            this.setPinyinTag(pinyinTag);
            this.setPinyin(pinyin);
   
    }

        public String getPinyin() {
            return pinyin;
        }

        public void setPinyin(String pinyin) {
            this.pinyin = pinyin;
        }

        public String getPinyinTag() {
            return pinyinTag;
        }

        public void setPinyinTag(String pinyinTag) {
            this.pinyinTag = pinyinTag;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public String getPhoneNumber() {
            return phoneNumber;
        }

        public void setPhoneNumber(String phoneNumber) {
            this.phoneNumber = phoneNumber;
        }

        public boolean isSelected() {
            return selected;
        }

        public void setSelected(boolean selected) {
            this.selected = selected;
        }

    }
 

#### 控制Loader的广播（ContactLoaderReceiver）

    public class ContactLoaderReceiver extends BroadcastReceiver {

        private ContactLoader mLoader;

        public ContactLoaderReceiver(ContactLoader mLoader) {
            this.mLoader = mLoader;

            /**
    
         * Register for events related to application installs/removals/updates.
             */
            IntentFilter filter = new IntentFilter(Intent.ACTION_PACKAGE_ADDED);
            filter.addAction(Intent.ACTION_PACKAGE_REMOVED);
            filter.addAction(Intent.ACTION_PACKAGE_CHANGED);
            filter.addDataScheme("package");
            mLoader.getContext().registerReceiver(this, filter);

            /**
             * Register for events related to sdcard installation.
             */
    
        IntentFilter sdFilter = new IntentFilter();
            sdFilter.addAction(Intent.ACTION_EXTERNAL_APPLICATIONS_AVAILABLE);
            sdFilter.addAction(Intent.ACTION_EXTERNAL_APPLICATIONS_UNAVAILABLE);
            mLoader.getContext().registerReceiver(this, sdFilter);
        }

        /**
         * This method is called when the BroadcastReceiver is receiving an Intent
         * broadcast.  During this time you can use the other methods on
         * BroadcastReceiver to view/modify the current result values.  This method
         * is always called within the main thread of its process, unless you
         * explicitly asked for it to be scheduled on a different thread using
         * thread you should
         * never perform long-running operations in it (there is a timeout of
         * 10 seconds that the system allows before considering the receiver to
         * be blocked and a candidate to be killed). You cannot launch a popup dialog
         * in your implementation of onReceive().
         * <p/>
         * <p><b>If this BroadcastReceiver was launched through a &lt;receiver&gt; tag,
         * then the object is no longer alive after returning from this
         * function.</b>  This means you should not perform any operations that
         * return a result to you asynchronously -- in particular, for interacting
         * with services, you should use
         * {@link android.content.Context#startService(android.content.Intent)} instead of
         * to interact with a service that is already running, you can use
         * {@link #peekService}.
         * <p/>
         * <p>The Intent filters used in {@link android.content.Context#registerReceiver}
         * and in application manifests are <em>not</em> guaranteed to be exclusive. They
         * are hints to the operating system about how to find suitable recipients. It is
         * possible for senders to force delivery to specific recipients, bypassing filter
         * resolution.  For this reason, {@link #onReceive(android.content.Context, android.content.Intent) onReceive()}
         * implementations should respond only to known actions, ignoring any unexpected
         * Intents that they may receive.
         *
         * @param context The Context in which the receiver is running.
         * @param intent  The Intent being received.
         */
        @Override
        public void onReceive(Context context, Intent intent) {
            /**
             * Tell the loader about the change.
             */
            mLoader.onContentChanged();
        }
    }

   
#### 联系人Loader（ContactLoader）

	public class ContactLoader extends AsyncTaskLoader<LinkedList<ContactInfo>> {

        /**
         * 获取库Phone表字段 *
         */
        private static final String[] PHONES_PROJECTION = new String[]{
                Phone.DISPLAY_NAME, Phone.NUMBER, Photo.PHOTO_ID, Phone.CONTACT_ID};

        /**
         * 联系人显示名称 *
         */
        private static final int PHONES_DISPLAY_NAME_INDEX = 0;

        /**
         * 电话号码 *
         */
        private static final int PHONES_NUMBER_INDEX = 1;

        private Context context;

        private ContactLoaderReceiver receiver;


        public ContactLoader(Context context) {
            super(context);
            this.context = context;
        }

        /**
         * Called on a worker thread to perform the actual load and to return
         * the result of the load operation.
         * <p/>
         * Implementations should not deliver the result directly, but should return them
         * from this method, which will eventually end up calling {@link #deliverResult} on
         * the UI thread.  If implementations need to process the results on the UI thread
         * they may override {@link #deliverResult} and do so there.
         * <p/>
         * To support cancellation, this method should periodically check the value of
         * <p/>
         * When the load is canceled, this method may either return normally or throw
         * call {@link #onCanceled} to perform post-cancellation cleanup and to dispose of the
         * result object, if any.
         *
         * @return The result of the load operation.
         * @see #onCanceled
         */
        @Override
        public LinkedList<ContactInfo> loadInBackground() {
            return this.getPhoneContacts(this.context);
        }

        /**
         * 得到手机通讯录联系人信息 *
         */
        private LinkedList<ContactInfo> getPhoneContacts(Context context) {
            LinkedList<ContactInfo> mContacts = new LinkedList<>();
            ContentResolver resolver = context.getContentResolver();
            // 获取手机联系人
            Cursor phoneCursor = resolver.query(Phone.CONTENT_URI,
                    ContactLoader.PHONES_PROJECTION, null, null, null);

            if (phoneCursor != null) {
                while (phoneCursor.moveToNext()) {

                    String phoneNumber = phoneCursor
                            .getString(ContactLoader.PHONES_NUMBER_INDEX);
                    if (TextUtils.isEmpty(phoneNumber)) {
                        continue;
                    }
                    String contactName = phoneCursor
                            .getString(ContactLoader.PHONES_DISPLAY_NAME_INDEX);
                    String pinyin = "" + CharacterParser.getInstance().getSelling(contactName);
                    String pinyin_tag;
                    if (!TextUtils.isEmpty(pinyin)) {
                        pinyin_tag = pinyin.substring(0, 1).toUpperCase();
                    } else {
                        pinyin_tag = "#";
                    }

                    mContacts.add(new ContactInfo(contactName, StringUtil.filterPhoneNumber(phoneNumber), pinyin_tag, pinyin));

                }
                phoneCursor.close();
            }

            /**
             * 排序
             */
            Collections.sort(mContacts, new Comparator<ContactInfo>() {
                @Override
                public int compare(ContactInfo lhs, ContactInfo rhs) {
                    if ("#".equals(lhs.getPinyinTag())) {
                        if (!"#".equals(rhs.getPinyinTag())) {
                            return 1;
                        }
                    } else {
                 c void onCanceled(LinkedList<ContactInfo> data) {
            super.onCanceled(data);
        }

        @Override
        protected void onReset() {
            // Ensure the loader is stopped
            onStopLoading();
            // The Loader is being reset, so we should stop monitoring for changes.
            if (this.receiver != null) {
                this.getContext().unregisterReceiver(this.receiver);
                this.receiver = null;
            }
        }

        @Override
        protected void onStopLoading() {
            cancelLoad();
        }       if ("#".equals(rhs.getPinyinTag())) {
                            return -1;
                        }
                    }
                    if (TextUtils.isEmpty(lhs.getPinyin())) {
                        return -1;
                    }
                    if (TextUtils.isEmpty(rhs.getPinyin())) {
                        return 1;
                    }
                    String l = lhs.getPinyin();
                    String r = rhs.getPinyin();
                    return l.compareTo(r);
                }
            });
            return mContacts;
        }

        @Override
        protected void onStartLoading() {
            super.onStartLoading();
            if (this.receiver == null) {
                this.receiver = new ContactLoaderReceiver(this);
            }
            forceLoad();
        }

        @Override
        public void onCanceled(LinkedList<ContactInfo> data) {
            super.onCanceled(data);
        }

        @Override
        protected void onReset() {
            // Ensure the loader is stopped
            onStopLoading();
            // The Loader is being reset, so we should stop monitoring for changes.
            if (this.receiver != null) {
                this.getContext().unregisterReceiver(this.receiver);
                this.receiver = null;
            }
        }

        @Override
        protected void onStopLoading() {
            cancelLoad();
        }
	}


#### Activity

    public class ImportContactActivity extends BaseActivity implements LoaderManager.LoaderCallbacks<LinkedList<ContactInfo>>{
        ......
        ......
        ......
        @Override
        public Loader<LinkedList<ContactInfo>> onCreateLoader(int i, Bundle bundle) {
            return new ContactLoader(this);
        }

        @Override
        public void onLoadFinished(Loader<LinkedList<ContactInfo>> linkedListLoader, LinkedList<ContactInfo> contactInfos) {
            if (contactInfos != null && contactInfos.size() != 0) {
                if (!this.load) {
                    this.contactList = contactInfos;
                    this.lvAdater.setList(this.contactList);
                    this.lvAdater.notifyDataSetChanged();
                    sideBar.setSections((String[]) lvAdater.getSections());
                    sideBar.invalidate();
                    this.load = true;
                }
            }
        }

        @Override
        public void onLoaderReset(Loader<LinkedList<ContactInfo>> linkedListLoader) {
            this.lvAdater.clearList();
        }

    }