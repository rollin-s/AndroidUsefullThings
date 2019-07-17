# AndroidUsefullThings

There is 2 important files. 
The TabBarActivity, and the Fragment that we want to show.

# DetailRVTabBarActivity

```
public class DetailRVTabBarActivity extends AppCompatActivity {

    private static final String TAG = "DetailRVTabBarActivity";

    // All the fragment that we want to show in the navigationView
    // We add one more fragment that will be the currently Shown fragment: (needed to detach from the FragmentManager)
    private DetailRVDetailFragment mDetailFragment;
    private DetailRVDocumentsFragment mDocumentFragment;
    private DetailRVUploadFragment mUploadFragment;
    private Fragment mCurrentFragment;

    // the 3 boolean values that will be used by changeTransactionFragment method.
    // The method needs to know if the fragment was showed before to prevent it from instanciating again
    private boolean wasDetailAdded = false;
    private boolean wasDocumentAdded = false;
    private boolean wasUploadAdded = false;

    @BindView(R.id.detail_rv_tab_bar) BottomNavigationView detail_rv_tab_bar;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        setContentView(R.layout.activity_tab_bar_detail_rv);
        ButterKnife.bind(this);


        // Setup all the tab Bar Buttons to link them with the correct fragment
        detail_rv_tab_bar.setOnNavigationItemSelectedListener(new BottomNavigationView.OnNavigationItemSelectedListener() {
            @Override
            public boolean onNavigationItemSelected(@NonNull MenuItem menuItem) {

                switch (menuItem.getItemId()) {
                    case R.id.detail_rv_details:
                        showDetail();
                        break;
                    case R.id.detail_rv_documents:
                        showDocuments();
                        break;
                    case R.id.detail_rv_upload:
                        showUpload();
                        break;
                }

                return true;
            }
        });

        showDetail();
    }

    private void showDetail() {
        if (mDetailFragment == null) {
            mDetailFragment = DetailRVDetailFragment.newInstance(mRvId);
        }

        Fragment oldCurentFragment = mCurrentFragment;
        mCurrentFragment = mDetailFragment;

        this.changeTransactionFragment(mDetailFragment, oldCurentFragment, wasDetailAdded);
        wasDetailAdded = true;
    }

    private void showDocuments() {
        if (mDocumentFragment == null) {
            mDocumentFragment = DetailRVDocumentsFragment.newInstance(mRvId);
        }

        Fragment oldCurentFragment = mCurrentFragment;
        mCurrentFragment = mDocumentFragment;

        this.changeTransactionFragment(mDocumentFragment, oldCurentFragment, wasDocumentAdded);
        wasDocumentAdded = true;
    }

    private void showUpload() {
        if (mUploadFragment == null) {
            mUploadFragment = DetailRVUploadFragment.newInstance(mRvId);
        }

        Fragment oldCurentFragment = mCurrentFragment;
        mCurrentFragment = mUploadFragment;

        this.changeTransactionFragment(mUploadFragment, oldCurentFragment, wasUploadAdded);
        wasUploadAdded = true;
    }


    /**
     * Generic method to update the fragmentManager with the correct fragment.
     * This method will add the fragment first to the FragmentManager, and then if it was added before, it will only attach it to the view without re-instanciate the view.
     * @param toReplace The fragment that we want to show
     * @param fromReplace The fragment that is curently showing
     * @param wasAlreadyAdded The boolean that tells if the fragment was already showed
     */
    private void changeTransactionFragment(Fragment toReplace, Fragment fromReplace, boolean wasAlreadyAdded) {
        if (fromReplace == null) {
          startTransactionFragment(toReplace);
        } else if (fromReplace.isVisible()) {
            getSupportFragmentManager().beginTransaction().detach(fromReplace).commit();

            if (wasAlreadyAdded) {
                getSupportFragmentManager().beginTransaction().attach(toReplace).commit();
            } else {
                getSupportFragmentManager().beginTransaction().add(R.id.detail_rv_frame_layout, toReplace).commit();
            }
        }
    }

    /**
     * Generic method used to replace a fragment in a fragmentManager. In our case, it is only used when it's the first fragment to be presented.
     * @param fragment
     */
    private void startTransactionFragment(Fragment fragment) {
        if (!fragment.isVisible()){
            getSupportFragmentManager().beginTransaction()
                    .replace(R.id.detail_rv_frame_layout, fragment).commit();
        }
    }
}
```

# DetailRVDocumentFragment

```
ublic class DetailRVDocumentsFragment extends BaseFragment {

    private static final String TAG = "DetailRVDocumentsFragme";

    @BindView(R.id.list_documents)
    ListView list_documents;

    private DetailRVDocumentsViewModel vm;
    private ListDocumentRvAdapter mAdapter;
    private List<Document> mList_doc;



    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Log.d(TAG, "onCreate is gonna be called the first time the fragment will be showed, but not the next time");

        // Instanciate the ViewModel only at the creation of the fragment.
        // The view model won't be instanciate and created again when the fragment will be attached
        vm = ViewModelProviders.of(this).get(DetailRVDocumentsViewModel.class);
        vm.init(getContext(), mRvId);

        final Observer<List<Document>> observer_list_doc = list_doc -> {
            // Add all the new objects to the current list of elements we want to show
            if (mList_doc == null) { mList_doc = list_doc; }
            mList_doc.removeAll(list_doc);
            mList_doc.addAll(list_doc);

            // If the fragment was just instanciate, create the adapter. If not, we just need to notifyDataSetChanged since we don't change the list object (we ust add the new objects in the list9.
            if (mAdapter == null) {
                mAdapter = new ListDocumentRvAdapter(getContext(), mList_doc);
                list_documents.setAdapter(mAdapter);
            } else {
                mAdapter.notifyDataSetChanged();
            }


        };

        if (!vm.getDocument_rv().hasObservers()) {
            vm.getDocument_rv().observe(this, observer_list_doc);
        }
    }

    @Override
    protected int getFragmentLayout() {
        return R.layout.fragment_detail_rv_documents;
    }

    @Override
    protected void onBaseActivityCreated() {
        Log.d(TAG, "ActivityCreated will be called each time the fragment is showed");

    }

    public static DetailRVDocumentsFragment newInstance(int rv_id) {

        Bundle args = new Bundle();

        DetailRVDocumentsFragment fragment = new DetailRVDocumentsFragment();
        args.putInt(KEY_RV_ID, rv_id);
        fragment.setArguments(args);
        return fragment;
    }

    @Override
    protected void onBaseCreateView() {
        Log.d(TAG, "CreateView is gonna be called each time the fragment is showed");

        // Each time the fragment is attached, we specify again the adapter for the current list
        if (mAdapter != null){
            list_documents.setAdapter(mAdapter);
            mAdapter.notifyDataSetChanged();
        }
    }

}
```


