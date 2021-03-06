EXE=CFortranTranslator
CXX=g++

CFLAGS_NO = -fno-rtti -fno-exceptions 
cov_comp = -fprofile-arcs -ftest-coverage -fno-inline
cov_lnk = -fprofile-arcs -ftest-coverage --coverage -fno-inline 
NO_WARN = -Wno-unused-label -Wno-unused-variable -Wno-char-subscripts -Wno-unused-but-set-variable -Wno-unused-value 
NO_WARN_DEBUGING = -Wno-format-security -Wno-sign-compare -Wno-format-zero-length
CFLAGS = -Wall -DPOSIX -g -O3 -fpermissive -fPIC -std=c++17 $(NO_WARN) $(NO_WARN_DEBUGING)
OBJ_EXT = o

SRC_ROOT = ..

include objs.mk

all: $(EXE) lib

$(EXE): out $(EXE_ROOT)
	cp ./CFortranTranslator ../bin/CFortranTranslator

install: 
	sudo cp -rf $(SRC_ROOT)/for90std/ /usr/local/include
	sudo cp -rf $(EXE_ROOT)/$(EXE) /usr/bin/$(EXE)
	sudo cp -rf $(EXE_ROOT)/for90std.a /usr/local/lib/for90std.a
	
lib: $(OBJ_FOR90)
	ar rvs $(EXE_ROOT)/for90std.a $(OBJ_FOR90)

out: CFLAGS_COV = 
out: make_dir gen_bison $(OBJS) 
	$(CXX) $(CFLAGS) -o $(EXE) $(OBJS) -L/usr/lib/ -lboost_system -lboost_exception

build_cov: CFLAGS_COV = -fprofile-arcs -ftest-coverage 
build_cov: make_dir gen_bison $(OBJS) 
	$(CXX) $(CFLAGS) $(CFLAGS_COV) --coverage -o $(EXE) $(OBJS) -L/usr/lib/ -lboost_system -lboost_exception

lib_cov: CFLAGS_COV = -fprofile-arcs -ftest-coverage 
lib_cov: $(OBJ_FOR90)
	ar rvs $(EXE_ROOT)/for90std.a $(OBJ_FOR90)

test_fortran:
	python $(SRC_ROOT)/demos/merge_test.py $(SRC_ROOT)/demos/fortran/ $(SRC_ROOT)/demos/merge n
	python $(SRC_ROOT)/demos/merge_test.py $(SRC_ROOT)/demos/fortran77/ $(SRC_ROOT)/demos/merge y
	./CFortranTranslator -fF $(SRC_ROOT)/demos/merge.for.test
	python $(SRC_ROOT)/demos/merge_test.py $(SRC_ROOT)/demos/snippet/ $(SRC_ROOT)/demos/merge n
	./CFortranTranslator -fF $(SRC_ROOT)/demos/merge.for.test --tree 
	python $(SRC_ROOT)/demos/merge_test.py ../demos/error/ $(SRC_ROOT)/demos/merge n
	./CFortranTranslator -fF $(SRC_ROOT)/demos/merge.for.test

test_for90std: install
	cd $(SRC_ROOT)/demos/for90std && make clean
	cd $(SRC_ROOT)/demos/for90std && make MODE=$(MODE)
	cd $(SRC_ROOT)/demos/for90std && ./io
	cd $(SRC_ROOT)/demos/for90std && ./farray
	cd $(SRC_ROOT)/demos/for90std && ./filesys

run_cov: 
	make test_fortran
	make test_for90std MODE=$(MODE)
	cd ../demos/test/ make MODE=$(MODE)
	gcov $(OBJ_GRAMMAR_ROOT)/for90.tab.cpp
	gcov -r -o $(OBJ_FOR90_ROOT)/*.gcno
	for name in `ls -al . | awk '{print $$NF}'| grep '.gcov$$' `;do mv $$name $(OBJ_FOR90_ROOT)/;done
	gcov -r -o $(OBJ_MAIN_ROOT)/*.gcno
	for name in `ls -al . | awk '{print $$NF}'| grep '.gcov$$' `;do mv $$name $(OBJ_MAIN_ROOT)/;done
	gcov -r -o $(OBJ_GRAMMAR_ROOT)/*.gcno 
	for name in `ls -al . | awk '{print $$NF}'| grep '.gcov$$' `;do mv $$name $(OBJ_GRAMMAR_ROOT)/;done
	gcov -r -o $(OBJ_PARSER_ROOT)/*.gcno
	for name in `ls -al . | awk '{print $$NF}'| grep '.gcov$$' `;do mv $$name $(OBJ_PARSER_ROOT)/;done
	gcov -r -o $(OBJ_TARGET_ROOT)/*.gcno
	for name in `ls -al . | awk '{print $$NF}'| grep '.gcov$$' `;do mv $$name $(OBJ_TARGET_ROOT)/;done

run_cov2: $(GCOVS)
$(GCOVS): test_fortran test_for90std
	gcov -r -o "$(patsubst $(GCOV_ROOT)%,$(OBJ_ROOT)%,$(@D))" "$(patsubst $(GCOV_ROOT)%,$(SRC_ROOT)%,$(@D))/$(patsubst %gcov,%cpp,$(@F))"
#	if [ -f "./$(@F)" ]; then mv "./$(@F)" $@; fi

run_lcov: $(OBJ_ROOT)
	lcov -c -o CFortranTranslator.lcov.info -d $(OBJ_ROOT)/
	genhtml CFortranTranslator.lcov.info -o CFortranTranslatorLCovHTML

run_coveralls_local: 
	make test_fortran
	make test_for90std MODE=$(MODE)
	cd ../demos/test/ make MODE=$(MODE)
	coveralls -b ./ -r ../ --exclude ./for90std --exclude ./demos/for90std --exclude ./demos/test --exclude ./src/grammar/for90.tab.cpp --exclude ./src/grammar/for90.flex.cpp --dryrun  --verbose --gcov-options '\-r'

run_coveralls: 
	make test_fortran
	make test_for90std MODE=$(MODE)
	cd ../demos/test/ make MODE=$(MODE)
	coveralls -b ./ -r ../ --exclude ./for90std --exclude ./demos/for90std --exclude ./demos/test --exclude ./src/grammar/for90.tab.cpp --exclude ./src/grammar/for90.flex.cpp --gcov-options '\-r'

tests: build_cov run_cov

$(OBJ_FOR90_ROOT)/%.$(OBJ_EXT): $(SRC_ROOT)/for90std/%.cpp 
	$(CXX) -c $(CFLAGS) $(CFLAGS_COV) $< -o $@
$(OBJ_MAIN_ROOT)/%.$(OBJ_EXT): $(SRC_ROOT)/src/%.cpp 
	$(CXX) -c $(CFLAGS) $(CFLAGS_COV) $< -o $@
$(OBJ_GRAMMAR_ROOT)/%.$(OBJ_EXT): $(SRC_ROOT)/src/grammar/%.cpp 
	$(CXX) -c $(CFLAGS) $(CFLAGS_COV) $< -o $@
$(OBJ_GRAMMAR_ROOT)/for90.tab.$(OBJ_EXT): $(SRC_ROOT)/src/grammar/for90.tab.cpp 
	# this is work around, for errors only in travis ci(build 21)
	$(CXX) -c $(CFLAGS) $(CFLAGS_COV) $(SRC_ROOT)/src/grammar/for90.tab.cpp -o $(OBJ_GRAMMAR_ROOT)/for90.tab.$(OBJ_EXT)
$(OBJ_PARSER_ROOT)/%.$(OBJ_EXT): $(SRC_ROOT)/src/parser/%.cpp 
	$(CXX) -c $(CFLAGS) $(CFLAGS_COV) $< -o $@
$(OBJ_TARGET_ROOT)/%.$(OBJ_EXT): $(SRC_ROOT)/src/target/%.cpp
	$(CXX) -c $(CFLAGS) $(CFLAGS_COV) $< -o $@

gen_bison: $(SRC_ROOT)/src/grammar/for90.y
	bison -d -v $(SRC_ROOT)/src/grammar/for90.y -o $(SRC_ROOT)/src/grammar/for90.tab.cpp
	mv $(SRC_ROOT)/src/grammar/for90.tab.hpp $(SRC_ROOT)/src/grammar/for90.tab.h

make_dir:
	mkdir -p $(EXE_ROOT)
	mkdir -p $(OBJ_FOR90_ROOT)
	mkdir -p $(OBJ_MAIN_ROOT)
	mkdir -p $(OBJ_GRAMMAR_ROOT)
	mkdir -p $(OBJ_PARSER_ROOT)
	mkdir -p $(OBJ_TARGET_ROOT)


.PHONY: clean_cov
clean_cov:
	find ./ -name "*.gcov" -delete
	rm -rf ./CFortranTranslatorLCovHTML

.PHONY: clean
clean:
	rm -rf $(OBJ_ROOT)
	find ./ -name "*.gcda" -delete
	find ./ -name "*.gcno" -delete
	find ../ -name "core" -delete
	cd ../demos/test && make clean
	cd ../demos/for90std && make clean
